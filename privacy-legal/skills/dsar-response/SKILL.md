---
name: dsar-response
description: >
  处理数据主体访问请求（或删除、可移植性、更正请求）
  并起草回复——验证身份、逐系统定位数据、评估豁免、起草确认
  和实质性回复信。当 DSAR 到来时、用户粘贴访问/删除/可移植性/更正
  请求或说"DSAR came in"、"access request"、"right to be forgotten"或
  "someone wants their data"时使用。
argument-hint: "[paste the request, or describe it]"
---

<!--
This file is a Chinese translation of the original by Anthropic PBC.
Original: https://github.com/anthropics/claude-for-legal
Licensed under Apache License 2.0
-->


# /dsar-response

1. 加载 `~/.claude/plugins/config/claude-for-legal/privacy-legal/CLAUDE.md` → DSAR 流程（系统列表、验证方法、SLA）。
2. 运行以下工作流。
3. 分类请求类型。检查升级触发条件——如果触发任何条件，在继续之前路由。
4. 处理：验证身份 → 遍历系统列表 → 豁免分析 → 起草。
5. 输出回复草案。不要发送——由人工审查并发送。
6. 根据内部流程记录 DSAR。

**在粘贴请求之前：** 请求将包含数据主体的 PII。确认你的会话和输出存储满足你的数据处理要求。编辑你不需要的内容（ID 附件、无关电子邮件线程）。不要在文件名中存储主体姓名。

```
/privacy-legal:dsar-response
[paste the request email]
```

---

# DSAR 响应起草

## 事项上下文

**事项上下文。** 检查执业级 CLAUDE.md 中的 `## Matter workspaces`。如果 `Enabled` 为 `✗`（内部用户的默认值），跳过本段的其余部分——skills 使用执业级上下文，事项机制不可见。如果已启用且没有活跃事项，询问："这是哪个事项的？Run `/privacy-legal:matter-workspace switch <slug>` 或说 `practice-level`。"加载活跃事项的 `matter.md` 以获取事项特定上下文和覆盖。将输出写入事项文件夹 `~/.claude/plugins/config/claude-for-legal/privacy-legal/matters/<matter-slug>/`。除非 `Cross-matter context` 为 `on`，否则永远不要阅读另一个事项的文件。

---

## 目的

DSAR 有截止日期（由适用制度设定）、流程（验证、定位、评估豁免、响应），以及很多可能出错的地方。此 skill 逐步完成每个步骤并起草响应。

## 司法管辖区假设

此分析假设您的配置中指定的司法管辖区范围。隐私规则、响应截止日期和合法依据因司法管辖区（GDPR 与州消费者隐私法与行业特定）而有很大差异。如果数据主体、处理活动或控制者位于与配置不同的司法管辖区，则此分析可能不适用于书面内容。

## 加载流程

读取 `~/.claude/plugins/config/claude-for-legal/privacy-legal/CLAUDE.md` → `## DSAR process`。该部分有：
- 系统列表（用户数据所在的每个位置）
- 身份验证方法
- 响应 SLA
- 谁处理例行事项与谁获得升级

如果系统列表为空或过时，标记它——不知道在哪里查找就无法完成完整的 DSAR。

## 工作流

### 步骤 1：分类请求

识别数据主体正在援引的权利。常见类别：

- **访问**——其数据副本 + 关于处理的信息
- **删除/擦除**——删除其数据（受豁免限制）
- **可移植性**——以机器可读格式呈现其数据
- **更正/更正**——修正不准确数据
- **反对**——停止特定处理（通常是营销）
- **限制**——在争议期间暂停处理
- **退出销售/共享/自动决策**——制度特定权利

**在继续之前研究适用规则。** 对于每个援引的权利，识别适用法律的司法管辖区（GDPR、英国 GDPR、CCPA/CPRA、其他美国州隐私法、行业制度）。引用控制法规或条例并附带精确引用——具体条款/章节、权利的范围、任何例外。注意生效日期；数据主体权利经常修订（每个立法会议的新州法律）。标记不确定性并升级以供律师验证，而不是陈述你未确认的规则。

> **没有静默补充。** 如果对配置的法律研究工具的搜索查询针对司法管辖区的权利、豁免或截止日期返回很少或没有结果，报告发现的内容并停止。不要在未询问的情况下从网络搜索或模型知识填充空白。说："搜索从 [工具] 返回了 [N] 个结果。对于 [制度/权利] 的覆盖范围似乎很薄。选项：（1）扩大搜索查询，（2）尝试不同的研究工具，（3）搜索网络——结果将标记为 `[web search — verify]`，在依赖之前应根据主要来源进行检查，或（4）标记为未验证并停止。你想要哪一个？"律师决定是否接受较低置信度的来源。
>
> **来源归因分层。** 用其来源标记每个引用。对于模型知识引用，使用三个层级之一，而不是单一的全局"verify"标记：
>
> - `[settled]`——稳定、众所周知的法定和监管参考，不太可能改变（例如，GDPR Art. 33、CCPA § 1798.100、FTC Act § 5、根据 § 1798.130(a)(2) 的 45 天 CCPA 响应窗口作为概念）。在提交前仍需验证，但优先级较低。
> - `[verify]`——真实但应验证的模型知识引用：具体实施法规、机构指导、案例裁决、阈值、生效日期、2023 年后修正案。
> - `[verify-pinpoint]`——精确引用（具体小节字母、卷/页码、段落号、监管子部分参考）具有最高的捏造风险，应始终根据主要来源进行验证。
>
> 工具检索的引用保留其来源标记（`[Westlaw]`、`[issuing authority site]` 或 MCP 工具名称）；网络搜索引用保持为 `[web search — verify]`；用户提供的引用保持为 `[user provided]`。分层显示真正的验证工作——验证一切的读者什么都没有验证。永远不要剥离或折叠标记。

有些请求是组合——"删除我的账户并先发送我的数据"是删除 + 可移植性。作为两个链接请求处理。

### 步骤 2：验证身份

根据 `~/.claude/plugins/config/claude-for-legal/privacy-legal/CLAUDE.md` 中的方法。常见方法：

- **登录验证：** 请求来自已验证会话 → 身份已确认
- **电子邮件匹配：** 请求来自存档电子邮件 → 对于低风险请求通常足够
- **额外验证：** 对于高价值账户或删除请求 → 质询问题、电话验证、ID 文档

**根据风险校准。** 过度验证将 DSAR 流程变为障碍（在监管机构面前形象不佳）。验证不足风险将他人的数据交给欺诈者。

如果无法验证身份：

```markdown
We were unable to verify that this request came from the individual whose data
is at issue. To proceed, please [verification step]. We cannot provide personal
data in response to a request we cannot verify.
```

这暂停时钟（可以说），但不要拖延——在几天内回应说你需要验证，而不是在第 29 天。

### 步骤 3：定位数据

从 `~/.claude/plugins/config/claude-for-legal/privacy-legal/CLAUDE.md` 遍历系统列表。对于每个系统：

| 系统 | 已查询？ | 找到数据？ | 什么 |
|---|---|---|---|
| 生产数据库 | | | |
| 分析（例如 Mixpanel、Amplitude） | | | |
| 支持票据（例如 Zendesk） | | | |
| CRM（例如 Salesforce、HubSpot） | | | |
| 电子邮件营销（例如 Marketo） | | | |
| 日志 | | | |
| 备份 | | |（注：通常豁免删除——见下文）|
| 第三方处理者 | | |（他们可能需要被通知删除）|

对于 B2B 处理者："数据主体"通常是*你客户的*最终用户。检查这实际上是你的客户要处理的 DSAR，而不是你的。许多处理者 DPA 说"将 DSAR 转发给控制者。"

### 步骤 4：豁免分析

并非所有内容都会被提供或删除。**在继续之前研究适用规则。** 对于每个项目，识别范围内制度可能适用的每个豁免（例如，第三方隐私、特权、商业秘密、安全、保留的法律义务、法律索赔的建立/辩护、交易必要性、备份轮换住宿、表达自由）。引用控制法规、条例或案例并附带精确引用。豁免范围因司法管辖区和制度而异——验证时效性并标记不确定性。

**不要在主观调用上缩小列表。** skill 在存在善意基础的地方提出豁免并标记不确定的豁免；律师在响应发出前缩小列表。删除后来证明适用的豁免代价高昂——一旦披露材料，豁免实际上就消失了。断言看似合理的豁免是正确的，律师可以在审查中更正。偏好可恢复的错误。

每个提议的豁免都有明确的注释：**"提议的——在断言之前需要律师审查。监管机构审查广泛的豁免主张，因此律师缩小此列表；skill 不缩小。"**

需要解决的常见反复问题：

- 记录是否包含需要在生产前编辑的关于*其他*人的数据？
- 是否有阻止删除的具体法律保留义务？引用它。
- 是否有针对此个人数据的活跃诉讼保全？
- 是否有需要记录的备份轮换或技术可行性住宿（不用作一般借口）？

**记录每个声称的豁免。** 如果监管机构问你为什么不删除某些内容，"我们有法律义务"需要引用。

### 步骤 5：起草回复——两封信

> **研究连接器飞行前检查。** 在发出任一信件或内部豁免分析之前，检查法律研究连接器在此会话中是否可达——Westlaw、EUR-Lex / 监管机构站点连接器或任何公司配置的研究 MCP。将此收集到审阅者备注中，根据 CLAUDE.md `## Outputs`——审阅者备注位于内部豁免分析和掩护备忘录上，而不是面向数据主体的外部 DSAR 信件上。如果没有连接器在步骤 1（正确分类）、步骤 4（豁免分析）或截止日期管理研究步骤中返回结果（或在运行时未配置），在内部审阅者备注的 **Sources:** 行中记录它——例如，`not connected — cites from training knowledge; claimed exemptions, response deadlines, and extension mechanisms are especially fabrication-prone, verify before asserting any exemption to a data subject or regulator`。每个引用的 `[model knowledge — verify]` 标记保持内联。不要在输出上方发出独立横幅。

大多数制度期望（或要求）将提示确认与实质性响应分开。两者都生成；不要将它们合并为一封等到第 45 天截止日期才发出的信。

- **步骤 5a ——确认信。** 在收到后几天内发送（目标：当天到 3-5 天，始终在制度法定窗口内）。确认收到，说明控制者理解的请求内容，说明响应时钟和目标日期，要求仍突出的任何身份验证材料。不包含实质性披露。提示确认是 DSAR 流程正常工作的第一个监管可见信号；它还降低了重复请求或早期投诉的风险。
- **步骤 5b ——实质性回复信。** 实际披露、删除确认或可移植性导出。在法定截止日期（或内部 SLA 如果更紧）之前发出。仅在身份验证完成和步骤 3 / 步骤 4 数据定位 + 豁免分析完成后。

**在向数据主体发送任一信件之前：** 阅读 `~/.claude/plugins/config/claude-for-legal/privacy-legal/CLAUDE.md` 中的 `## Who's using this`。如果角色为非律师：

> 发送 DSAR 响应有法律后果——内容、声称的豁免和遗漏都可供监管机构审查，误述会成为执法风险。你是否已与律师审查此内容？如果是，继续。如果不是，这是带给他们的简报：
>
> [生成 1 页摘要：数据主体、援引的权利、适用制度、跨系统列表定位的内容、根据哪些豁免扣留的内容、身份验证姿态、响应截止日期以及在信件发出前要问律师的三件事。]
>
> 如果你需要在你的司法管辖区找到执业律师、事务律师、大律师或其他授权法律专业人士：你的专业监管机构的推荐服务是最快的起点（美国的州律师协会、英格兰和威尔士的 SRA/律师标准委员会、苏格兰/北爱尔兰/爱尔兰/加拿大/澳大利亚的律师协会，或你司法管辖区的同等机构）。

在没有明确是的情况下不要越过此关卡。

> **注意：** 两封 DSAR 信件都是发送给数据主体的外部可交付成果。**不要**在任一信件上包含来自 `~/.claude/plugins/config/claude-for-legal/privacy-legal/CLAUDE.md` `## Outputs` 的工作产品标题。伴随信件的内部注释、日志和豁免分析是律师工作产品——将它们分开，并根据 `~/.claude/plugins/config/claude-for-legal/privacy-legal/CLAUDE.md` `## Outputs` 在其前面添加工作产品标题（根据用户角色而不同——见 `## Who's using this`）。
>
> **在发送任一信件之前：** 这是供律师审查的草案，不是要发送的响应。发送使控制者承诺立场，可能放弃豁免，并可能启动监管机构的时钟。执业律师在任一信件发给数据主体之前审查、编辑和批准。不要发送未经审查的信件。

#### 步骤 5a ——确认信模板

```markdown
Subject: We received your privacy request — [Company] — [date]

Dear [Name],

We received your [access / deletion / portability / correction] request on [date received].

**Your request, as we understand it:** [一句话重述——例如，"a copy of all personal data we hold associated with your account, along with the categories of third parties with whom we share it, and deletion of your account after we provide the copy."]

**What happens next:**
- Our target date for the substantive response is [date — no later than the regime's statutory deadline; use internal SLA if tighter]. [如果身份验证未完成："We need [specific verification step] before we can proceed — see below."]
- If we need more time because the request is complex or we receive other requests from you at the same time, we will tell you before the initial deadline and explain why. [如果制度允许延期，引用控制条款。]
- No fee applies to this request. [或者：费用仅在制度允许且请求明显没有根据或过度时适用——引用条款。]

[如果身份验证未完成：]
**To verify your identity,** please [specific verification step — 例如，reply to this email from the address on file with the last 4 digits of the payment method we have on file]. This does not pause our deadline; we continue to work in parallel.

If you have questions, contact [privacy contact].

[Sender]
```

**时钟开始规则。** 响应时钟在收到请求时开始，而不是在完成身份验证时——除非适用制度另有规定。不要在验证时默认暂停时钟。如果制度有不同的触发条件，引用它；不要假设。

#### 步骤 5b ——实质性回复信模板

**访问请求响应：**

```markdown
Subject: Your Data Access Request — [Company] — [date]

We received your request on [date] for a copy of the personal data we hold about you.

**What we found:**

We hold the following categories of personal data associated with [identifier]:

| Category | Source | Purpose | Retained until |
|---|---|---|---|
| [Account info: name, email] | You, at signup | Account management | Account deletion |
| [Usage data] | Our service | Analytics, product improvement | [period] |
| [Support correspondence] | You | Customer support | [period] |

**Your data is attached** in [format]. [Secure delivery note — password-protected
archive, secure link with expiry, etc.]

**Third parties:** We share data with the following processors: [list or link to
subprocessor page].

**Your other rights:** You may also request [deletion / correction / portability].
To do so, [method].

**Data we did not include:**
- [Category] — [exemption and reason, 例如，"internal security logs — disclosure
  would compromise security measures"]
- [Data about other individuals has been redacted from support correspondence]

If you have questions about this response, contact [privacy contact].
```

**删除请求响应：**

```markdown
Subject: Your Deletion Request — [Company] — [date]

We received your request on [date] to delete the personal data we hold about you.

**What we deleted:**

| Category | System | Deleted on |
|---|---|---|
| [Account and profile] | Production | [date] |
| [Analytics events] | [Amplitude/etc.] | [date] |
| [etc.] | | |

**What we retained and why:**

| Category | Reason | Retained until |
|---|---|---|
| [Transaction records] | Legal obligation (tax record retention, [cite law]) | [date] |
| [Backup snapshots] | Will be deleted on next rotation | [date] |

**Third-party processors:** We have instructed [list] to delete your data from
their systems.

Your account is now closed. If you have questions, contact [privacy contact].
```

### 步骤 6：记录

DSAR 会接受审计。记录：
- 收到日期
- 身份验证日期
- 响应日期
- 提供/删除的内容
- 声称的豁免及其依据
- 处理人

如果你的团队使用 DSAR 跟踪工具，在那里创建记录。如果没有，日志文件可以。

## 升级触发条件

根据 `~/.claude/plugins/config/claude-for-legal/privacy-legal/CLAUDE.md` → 升级表，在以下情况升级：
- 请求者是（或可能是）原告、对方律师或记者
- 请求范围异常（"all data including internal communications about me"）
- 对此个人数据有诉讼保全（删除请求 + 诉讼保全 = 冲突，律师决定）
- 请求者正在争议先前的 DSAR 响应
- 任何监管机构被抄送或提及

## 截止日期管理

**两封信规则。** 每个 DSAR 生成确认信（提示——目标为收到后当天到 3-5 天）和实质性回复信（按法定截止日期）。大多数制度要求或期望将提示确认与实质性响应分开；在第 45 天发送的单一组合信即使实质正确也是流程失败。

**研究特定援引权利和适用司法管辖区的当前有效响应截止日期。** 检查延期机制是否存在、它购买多少额外时间以及数据主体必须收到什么通知以调用它。识别时钟何时开始（收到 vs 验证 vs 其他触发——默认规则是收到；按制度验证）。引用控制法规或条例并附带精确引用。注意生效日期——数据保护响应时间线经常修订，新州法律引入自己的时钟。

如果 `~/.claude/plugins/config/claude-for-legal/privacy-legal/CLAUDE.md` → `## DSAR process` 记录了比法律截止日期更紧的内部 SLA，使用内部 SLA 并记录法律支持。

如果你需要延期，在第一个截止日期之前发送"我们需要更多时间"通知。当天的延期看起来不好。

## 此 skill 不做什么

- 它不直接查询系统。它带你通过检查清单；人工（或连接的工具）进行实际查询。
- 它不在接近的案件上做出豁免调用。它将它们标记给律师。
- 它不发送响应。起草、审查、人工发送。
