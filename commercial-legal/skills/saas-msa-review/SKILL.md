---
name: saas-msa-review
description: >
  参考：审查 SaaS 订阅协议，重点关注订阅交易中最重要的条款——自动续约机制、价格上涨机制、
  数据可携权、正常运行时间 SLA 以及 subprocessor 权利。当检测到 SaaS 或订阅协议时，由
  /commercial-legal:review 加载。
user-invocable: false
---

<!--
This file is a Chinese translation of the original by Anthropic PBC.
Original: https://github.com/anthropics/claude-for-legal
Licensed under Apache License 2.0
-->


# SaaS / 订阅协议审查

## 事项上下文

**事项上下文。** 检查执业级 CLAUDE.md 中的 `## Matter workspaces`。如果 `Enabled` 为 `✗`（内部用户的默认设置），跳过本段的其余部分——skills 使用执业级上下文，事项机制不可见。如果已启用且没有活跃事项，询问："这是哪个事项的？运行 `/commercial-legal:matter-workspace switch <slug>` 或说 `practice-level`。"加载活跃事项的 `matter.md` 以获取事项特定上下文和覆盖。将输出写入事项文件夹 `~/.claude/plugins/config/claude-for-legal/commercial-legal/matters/<matter-slug>/`。除非 `Cross-matter context` 为 `on`，否则永远不要阅读另一个事项的文件。

---

## 目的

SaaS 协议的风险特征与一次性供应商合同不同。美元在续约时复利，数据不断积累，切换成本每月增长。此 skill 带着这种认识进行审查。

它运行 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 中的标准剧本检查，并在订阅交易中最棘手的条款上添加 SaaS 特定的叠加层。

## 司法管辖区假设

SaaS 条款（自动续约通知要求、价格上涨上限、数据可携性要求、subprocessor 规则）对司法管辖区敏感——加州、纽约和欧盟规则存在实质性差异，一些州有自动续约法规会覆盖私人合同条款。此审查应用 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 中团队的立场，这些立场假设了那里记录的适用法律。如果协议选择了不同的适用法律，或交易跨越有法规覆盖的司法管辖区（例如，欧盟用户、加州消费者），标记它——分析可能不能按书面转移。

> **无静默补充。** 如果对配置的法律研究工具（Westlaw 或律所平台）的研究查询对可能影响交易的法规覆盖（自动续约法规、数据可携要求、消费者保护规则）返回很少或没有结果，报告发现的内容并停止。**不要**在未询问的情况下从网络搜索或模型知识中填补空白。说："搜索从 [工具] 返回了 [N] 个结果。对于 [司法管辖区/规则] 的覆盖似乎很薄。选项：(1) 扩大搜索查询，(2) 尝试不同的研究工具，(3) 搜索网络——结果将标记为 `[web search — verify]` 并在依赖前应根据主要来源检查，或 (4) 标记为未验证并停止。你想要哪个？"律师决定是否接受低置信度的来源。
>
> **来源归因。** 当审查引用法规、条例或案例时（例如，覆盖合同条款的州自动续约法），标记引用：`[Westlaw]`、`[statute / regulator site]`，或从法律研究连接器检索的引用的 MCP 工具名称；`[web search — verify]` 用于网络搜索引用；`[model knowledge — verify]` 用于从训练知识中回忆的引用；`[user provided]` 用于来自对手方草案或内部文件的引用。标记为 `verify` 的引用具有更高的伪造风险，应首先检查。永远不要删除或折叠标记。

## 加载剧本

**哪一方？** 在应用剧本之前，确定公司在此 SaaS 协议中的哪一方。通常很明显：如果对手方是向你销售平台的 SaaS 供应商，你是采购方。如果你是 SaaS 供应商且对手方是你的客户，你是销售方。如果不明显（转售安排、白标交易），询问："[公司]在此协议中是哪一方——供应商还是客户？"从配置中读取匹配的剧本部分（`### Sales-side playbook` 或 `### Purchasing-side playbook`）。在输出中注明哪一方，以便审查者知道应用了哪个剧本。如果匹配的一方是 `[Not configured]`，停止并告诉用户在此审查继续前运行 `/commercial-legal:cold-start-interview --side <side>`。

首先读取 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md`。匹配方的通用剧本（责任、赔偿、终止、适用法律）完全适用——运行 vendor-agreement-review skill 的所有标准检查。

然后查找 `## Playbook` → 匹配方 → `SaaS positions` 部分。那是团队记录其在自动续约通知窗口、可接受的价格上涨机制、数据导出权利、SLA 阈值、subprocessor 批准权利和废弃通知方面立场的地方。此 skill 不附带这些项目的默认值——正确的数字因交易规模、供应商谈判力和团队风险容忍度而异。

如果 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 没有涉及此审查中出现的 SaaS 特定条款，询问：

> 你的剧本没有涵盖 [条款——例如"最长可接受自动续约通知窗口"或"供应商保留匿名衍生数据是否可接受"]。你团队的立场是什么？我会将其添加到 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md`。

记录答案并继续。

## SaaS 特定叠加层

对于以下每个类别，列出你在合同中发现的内容，并与 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 中团队的立场进行比较。不要应用此 skill 中的硬编码阈值。

### 1. 自动续约机制

SaaS 交易出问题最常见的单一方式：没有人注意到续约通知窗口，我们又被锁定了一年，价格还更高。

检查每个元素并与 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 中团队的 `SaaS positions` 进行比较：

- **续约期限长度**（例如，与初始期限相同、更长、多年自动转换）
- **取消通知窗口**（续约前的天数）
- **通知方式**（电子邮件、书面通知给法律、仅通过门户、挂号信）
- **续约价格**（相同、CPI 上限、当时的现行定价、无上限酌情决定）

**提取并记录**确切的续约日期和通知窗口，无论是否有任何项目被标记。这会输入到 renewal-tracker skill。

### 2. 价格上涨

对照 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 检查每个元素：

- **年度上涨幅度**（固定%、CPI、无上限等）
- **超额使用定价**（公布的费率卡、溢价费率、未指定）
- **"费用"的范围**（仅订阅费与宽泛定义的"额外服务"）

### 3. 数据可携性和退出

当（而不是如果）我们离开这个供应商，我们能拿回数据吗？对照 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 检查每个元素：

- **导出格式**（开放/标准、专有但有文档、"商业上合理"）
- **导出可用性**（随时自助、期限内按请求、仅在终止时）
- **终止后访问**（终止后可导出的天数）
- **导出成本**（免费、按时间和材料计费、按 GB 或按记录计费）
- **删除认证**（按请求认证、无、供应商保留衍生数据）

供应商保留"匿名化"或"聚合"衍生数据是实质性立场——在 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 中确认团队的立场并无论如何标记。

### 4. 正常运行时间和 SLA

只有在业务真正依赖此服务正常运行时才重要。如果这是个可有可无的工具，跳过此部分——不要在调查工具的 SLA 上花费谈判资本。

对照 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 检查每个元素：

- **正常运行时间承诺**（百分比，或"商业上合理的努力"）
- **计量周期**（每月、每季、每年）
- **补救措施**（服务积分——如何计算、是否有上限、是否是唯一补救措施）
- **计划维护排除**（定义的窗口、提前通知、无限制）
- **积分作为唯一补救措施与责任上限的相互作用**

### 5. Subprocessor

这是数据保护问题，但它是 SaaS 特定的，因为 subprocessor 列表在订阅期内*会发生变化*。

对照 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 检查每个元素：

- **当前列表**（已发布、按请求、不可用）
- **变更通知**（提前通知期，或无）
- **异议权**（阻断、通知并终止、仅通知、无）

### 6. 服务变更和废弃

SaaS 供应商会更改其产品。通常没问题。有时他们会废弃你购买的功能。

对照 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 检查每个元素：

- **重大不利变更**（因重大降级而终止的权利、仅通知、不受限制）
- 团队依赖的功能的**废弃通知期**
- 替代品上的**功能对等**（相同价格层级、更高层级）



## AI 和机器学习权利

**AI/ML 数据权利决策程序。** 不要只检查 AI 训练条款是否存在。SaaS 合同中第一大新兴谈判点在结构上远不止一行存在性检查。逐步进行：

1. **明确授权。** 合同是否明确授予供应商使用客户数据/客户内容/使用数据进行 AI 训练、模型改进或 ML 开发的权利？采购方：这通常是否——客户数据训练供应商的模型意味着客户在补贴供应商的产品，可能还泄露了竞争信息。销售方：如果能得到这个，这是收入；如果滥用，是声誉风险。
2. **通过政策引用的隐性授权。** 合同是否通过引用纳入了供应商的隐私政策或服务条款？供应商能否通过单方面政策更新添加训练权利？检查："双方同意按时更新的提供商隐私政策"是等待发生的训练权利授权。还要注意"服务改进"或"分析"的兜底条款，以及将日志/遥测数据从客户数据定义中排除的"使用数据"定义，这样数据使用限制就不适用。
3. **匿名化标准。** 如果供应商声称只在"匿名化"或"聚合"数据上训练，标准是什么？没有定义的"匿名化"很弱。是否符合 GDPR Recital 26 / HIPAA Safe Harbor / 命名标准？可逆吗？
4. **竞争污染。** 供应商是否服务你的竞争对手？如果是，在你的数据上训练可能会将竞争情报泄露到你的竞争对手看到的输出中。是否有竞争隔离承诺？
5. **退出范围和持久性。** 如果有退出，它是涵盖所有 AI 用途还是只有一些？是否在续约和 TOS 更新后仍有效？是每个用户还是每个组织？许多供应商默认训练并提供隐藏在管理控制台中的退出——检查合同是否明确了默认设置。
6. **输出所有权。** 如果 SaaS 产品本身是 AI 生成的（起草、摘要、分析），谁拥有输出？供应商能否将你的输出用作训练样本？也检查第三方 AI subprocessor——供应商可能将客户数据发送给第三方 LLM（OpenAI、Anthropic、Google），subprocessor 列表/数据流就是它出现的地方。
7. **下游监管链。** 供应商使用你的数据进行 AI 是否对**你**产生监管风险？欧盟 AI 法案部署者义务、FTC §5 未披露数据共享风险（参见 *FTC v. Humor Rainbow/OkCupid*）、州 AI 法律。

将每个与剧本立场匹配。执业档案的 `## AI/ML training rights` 部分应该有每个维度的立场。如果协议对所有七个都保持沉默，这仍然是一个发现："协议对 AI/ML 训练权利保持沉默——请求明确禁止或与上述七个维度各自挂钩的定义排除。"

## 责任上限决策程序

**上限金额是上限中最不重要的部分。** 责任限制不是单一的"对照剧本检查"项目。逐步进行：

1. **直接与间接/后果性损害赔偿。** 上限是否适用于所有责任，还是只适用于直接损害？对直接损害赔偿有 12 个月上限但对后果性损害赔偿无上限，与 12 个月总上限是完全不同的立场。明确说明两种处理方式。

2. **上限基数——逐字引用。** "12 个月上限"可能意味着：(a) 索赔前 12 个月支付的费用，(b) 当前 12 个月期间应付的费用，(c) 过去 12 个月使用的费用，(d) 当前订单表单下的费用，(e) 曾经支付的总费用。这些可能相差一个数量级。引用精确的语言。如果模糊，标记它："上限基数是模糊的——`[引用的语言]`——可能意味着 [X] 或 [Y]。签署前确认。"

3. **上限-例外相互作用。** 对数据违约、IP 和保密有无上限赔偿的 10 万美元上限，对于 SaaS 争议中实际出现的索赔实际上是无上限的。列举上限之上的内容（例外），上限之下的内容（实际上限制的内容），并评估受限表面是否有意义："上限涵盖 [一般合同违约]。数据违约、IP 赔偿和保密被排除且无上限。对于此供应商的风险状况，受限表面是 [有意义的/名义上的]。"

4. **每个维度的你的剧本立场。** 执业档案应该有以下立场：直接上限（费用倍数）、间接损害赔偿（排除/上限/无上限）、例外列表（上限之上可接受什么）和上限基数（你将接受哪个定义）。如果剧本有一个"标准立场"字段，说明："你的剧本有一个单一的上限立场——考虑拆分为直接/间接/例外/基数以进行更精确的审查。"

## 司法管辖区差异检查

**剧本全球应用一个适用法律偏好。可执行性差异很大。** 在表面上接受剧本立场之前，对照主要差异检查 SaaS 合同的实际适用法律：

- **非招揽/非竞争：** 在加州不可执行（Bus. & Prof. Code §16600）。在许多欧盟司法管辖区受到限制。在其他地方有条件可执行。`[jurisdiction — verify]`
- **自动续约：** 加州 GBL §17600-17606、纽约 GBL §527-a、伊利诺伊州 815 ILCS 601 有特定的消费者/B2B 通知要求。其他州有所不同。`[jurisdiction — verify]`
- **责任排除：** 欧盟和英国不公平合同条款规则（UCTA 1977、消费者权利法案 2015）限制消费者排除。一些美国州限制排除重大过失或故意不当行为。`[jurisdiction — verify]`
- **赔偿：** 一些州使赔偿者自己的过失的赔偿无效。`[jurisdiction — verify]`
- **保密期限：** 一些司法管辖区将"永久"保密限制在合理期限内。`[jurisdiction — verify]`

当剧本立场与合同适用法律的可执行性冲突时，标记："你的剧本偏爱 [X]，但此合同受 [Y] 法律管辖，其中 [X] 是 [不可执行的/受限制的/受法规覆盖的]。`[jurisdiction — verify]`"

## 红线粒度

**以尽可能小的粒度编辑。** 红线是谈判产物，而非重写。整条条款替换发出信号"我们丢弃了你的起草"——这是激进的，迫使对手方重新阅读整个条款，并丢弃其起草中没问题的部分。外科红线——删除一个词、插入一个短语、重组一个子条款——发出"我们有具体要求"的信号，并且更快阅读、理解和接受。

默认为实现剧本立场的最小化编辑：
- 在短语之前替换一个**词**。（"twelve (12)" → "twenty-four (24)"）
- 在句子之前替换一个**短语**。（"paid by the Buyer" → "paid and payable by the Buyer"）
- 在替换句子之前重组一个**子条款**。（添加"(a)"和"(b)"以拆分复合条件。）
- 在替换条款之前替换一个**句子**。
- 只有当对手方版本与你的立场相差太远以至于外科编辑比新草案更难阅读时，才替换**整个条款**——当你这样做时，在传输中说明："我们替换了 §8.2 而不是标记它，因为更改很广泛。很高兴引导你了解差异。"

如有疑问，越小越好。收到外科红线的客户信任你仔细阅读。收到整批替换的客户怀疑你是否根本没读。

## 输出

使用 vendor-agreement-review 备忘录结构，在标准剧本检查后添加 SaaS 特定部分。vendor-agreement-review 备忘录已经携带特权抬头。

**双重严重性。** 每个 SaaS 特定发现都有两个轴（见 CLAUDE.md `## Dual severity`）：
- **法律风险：** 🔴 Critical | 🟠 High | 🟡 Medium | 🟢 Low
- **业务摩擦：** 🔴 Blocks deals | 🟠 Slows deals | 🟡 Confuses customers | 🟢 Invisible

数据退出、自动续约和价格上涨发现最有可能是 🟢 法律/🔴 业务——条款可执行，但它是客户无法离开或续约让财务意外的原因。以业务摩擦严重性而非法律严重性呈现这些。

```markdown
### 底线

[可以签署/先要为 X 争取/离开——一句话说明原因]

### AI 和机器学习权利

[第一大新兴 SaaS 谈判点。标记：明确的 ML 训练条款、"服务改进"兜底条款、使用数据定义、输出所有权、第三方 AI subprocessor、退出与选择加入。如果协议保持沉默："对 AI/ML 训练权利保持沉默——请求明确禁止或定义排除。"]

## SaaS 特定发现

### 自动续约
**续约日期：** [日期]
**通知窗口：** 在 [日期]前取消（续约前 [N] 天）
**续约价格机制：** [按书面]
**剧本符合度：** [在立场内/偏差/未涉及]
**为续约跟踪器标记：** [是——以及跟踪器需要的记录]

### 价格上涨
[发现对照 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 立场]

### 数据退出
[发现——这是业务负责人应该读的部分]

### SLA
[发现，或"跳过——服务对业务不关键，来自 [利益相关者]"]

### Subprocessor
[发现对照 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 立场]

### 服务变更
[发现对照 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 立场]
```

## 交接

**到 renewal-tracker：** 当你找到续约日期和通知窗口时，交接它们。renewal-tracker 登记册期望以下字段（完整 schema 见 `skills/renewal-tracker/references/renewal-register.yaml`）：

```yaml
counterparty:         [name]
agreement:            [title]
signed_date:          [ISO date]
initial_term_end:     [ISO date]
renewal_mechanism:    [e.g., "auto-renew annual"]
notice_period_days:   [integer]
cancel_by_effective:            [ISO date — initial_term_end minus notice_period_days]
price_on_renewal:     [mechanism as written]
annual_value:         [integer, if stated]
business_owner:       [email, if known]
clm_id:               [id if available]
status:               active
```

如果任何字段无法从合同或上下文确定，省略它并注明哪些字段缺失以便人工填写。`clm_id`、`annual_value` 和 `business_owner` 尤其可能需要人工输入。

**到 escalation-flagger：** 如果任何 SaaS 特定检查触及 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 中团队的"永不接受"或升级触发列表，escalation-flagger skill 进行路由。

## 关于该争取什么的说明

SaaS 供应商，尤其是大型供应商，谈判其文件的意愿与航空公司谈判机票条款的意愿差不多。**根据团队剧本**挑选要争取的——`~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 中的 `SaaS positions` 部分应该区分团队总是会推的条款、只在重要交易中争取的条款，以及会放过的条款。如果剧本没有划定这些界限，询问。

根据合同价值和切换成本校准。年度 5,000 美元且有简单替代方案的工具比我们将在其上构建的年度 50 万美元平台审查力度轻得多。

## 以下一步决策树结束

按照 CLAUDE.md `## Outputs` 以下一步决策树结束。根据此 skill 刚刚产生的内容自定义选项——五个默认分支（起草 X、升级、获取更多事实、观察等待、其他事情）是起点，而不是锁定。树就是输出；律师选择。
