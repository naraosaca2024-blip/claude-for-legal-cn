---
name: nda-review
description: >
  参考:对入站 NDA 进行快速分类为绿 / 黄 / 红,以便团队只将律师时间花在需要它的 NDA 上。为销售和 BD 构建,以便在 ping 法律之前自助服务。当检测到 NDA 时由 /commercial-legal:review 加载。
user-invocable: false
---

<!--
This file is a Chinese translation of the original by Anthropic PBC.
Original: https://github.com/anthropics/claude-for-legal
Licensed under Apache License 2.0
-->


# NDA 审查

## 事项上下文

**事项上下文。** 检查执业级 CLAUDE.md 中的 `## Matter workspaces`。如果 `Enabled` 为 `✗`,跳过本段的其余部分——skills 使用执业级上下文,事项机制不可见。如果已启用且没有活跃事项,询问:"这是哪个事项的? Run `/commercial-legal:matter-workspace switch <slug>` 或说 `practice-level`。"加载活跃事项的 `matter.md` 以获取事项特定上下文和覆盖。将输出写入事项文件夹 `~/.claude/plugins/config/claude-for-legal/commercial-legal/matters/<matter-slug>/`。除非 `Cross-matter context` 为 `on`,否则永远不要阅读另一个事项的文件。

---

## 目的地检查

在生成输出之前,检查它要去哪里。如果用户命名了目的地(频道、分发列表、对手方、"每个人"),询问它是否在特权圈内。公共频道、公司范围列表、对手方/对方律师、供应商和客户(对于工作产品)放弃保护。当目的地看起来在圈外时,标记它并提供(a)仅用于法律的特权版本,(b)用于更广泛频道的净化版本,或(c)两者——不要默默地应用特权标题然后帮助将其粘贴到标题不保护它的地方。请参阅此 plugin 的 CLAUDE.md 中的规范 `## Shared guardrails → Destination check`。

---

## 目的

大多数入站 NDA 都没问题。少数有地雷。此 skill 在不到一分钟内对它们进行分类,以便法律只阅读重要的内容。

**目标:** 绿色 NDA 应该只需要签名。黄色需要律师的眼睛关注一两个具体事项。红色在任何人在浪费时间之前停止。

## 首先加载剧本

**哪一方?** 在应用剧本之前,确定公司在此 NDA 中的哪一方。通常从上下文中明显:如果对手方是正在评估你产品的供应商或合作伙伴,你是销售方;如果你正在评估他们的,你是采购方。互惠 NDA 仍然有一方——是谁的文件,评估在哪个方向运行。如果不明显,询问。阅读配置中的匹配剧本部分(`### 销售方剧本`或`### 采购方剧本`)。在输出中注意哪一方,以便审查者知道应用了哪个剧本。如果匹配的一方是`[未配置]`,停止并告诉用户在分类继续之前运行 `/commercial-legal:cold-start-interview --side <side>`。

**在分类任何内容之前,阅读 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` → `## 剧本` → 匹配方 → `NDA 分类立场`。**该部分是*此*团队在*此*方上使 NDA 成为绿、黄或红的内容来源。此 skill 不带有 NDA 条款的默认立场——法律、市场和每个团队的风险偏好变化太大,硬编码默认值不安全。

如果 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 尚未拥有 `NDA 分类立场`部分,或者它对你正在审查的 NDA 中出现的条款保持沉默,询问用户:

> 你的剧本不涵盖[条款——例如,"residuals clauses"、"survival period"、"你是接收方的单向 NDA"]。你的默认立场是什么——什么时候应该是绿、黄、红?我会将其添加到 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md`,以便下次审查一致。

然后在 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 中记录答案,并使用新立场继续分类。

## 范围检查

**在审查 NDA 特定条款之前,检查文档是否做的比其名称建议的更多。** 互惠商业 NDA 可以隐藏:standstills、许可授予、排他性、非招揽、非竞争、IP 转让、优先购买权、最惠国条款以及管辖远不止保密争议的仲裁/管辖条款。

如果 NDA 包含保密以外的义务: **无论 NDA 条款分析如何,自动为黄。** 标记非 NDA 条款:

> 此文档标记为 NDA,但包含[standstill / 许可授予 / 非招揽 / 排他性 / IP 转让 / ROFR / MFN / 广泛仲裁]。这不仅仅是 NDA。路由供律师审查。

不要默默地推动标记为"NDA"的文档通过 NDA 分类,当实质性义务是服务协议、条款表或 NDA 包装中的契约包时。

## 分类

通过应用 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 中的立场,将 NDA 分类为三个存储桶之一。下面的存储桶定义是稳定的;填充每个存储桶的*标准*来自剧本。

### 绿——路由到签名

NDA 满足团队剧本中的每个立场,且没有条款根据剧本触发红旗。剧本通常涵盖的检查示例:互惠性、条款长度、存续期、例外、管辖法律、限制性契约、费用转移。在称绿之前,根据 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 确认每一个。

**GREEN 需要律师审查的剧本立场。** GREEN 是无需律师审查即可签名的唯一路径。它不能针对默认或缺失的立场发出。在发出 GREEN 之前,检查:执业档案是否有律师审查的 `## NDA 分类立场`部分?如果没有:

> 我无法在没有执业档案中的律师审查 NDA 立场的情况下发出 GREEN。运行 `/commercial-legal:cold-start-interview --full` 与你的商业法律顾问设置它们,或将此 NDA 路由供律师审查。针对默认值发出 GREEN 意味着非律师设置了下一个非律师依赖的立场。

不要针对默认值路由到签名。当立场缺失时,YELLOW 是正确的调用——它将 NDA 浮现给可以决定的人类。

**输出:**

在前面加上 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` `## Outputs` 中的工作产品标题(根据用户角色而不同——见 `## Who's using this`)。

```markdown
[WORK-PRODUCT HEADER — per plugin config ## Outputs]

## NDA 分类: [对手方]

绿——路由到签名

### 执行摘要

根据剧本未识别红旗。按标准流程路由以供签名。

| 检查 | 状态 | 剧本参考 |
|---|---|---|
| [每个剧本检查] | [通过/失败] | [`~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 部分] |

**下一步:** [提交到 [CLM] 标准 NDA 工作流程 | 发送给[来自 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 的审批人]以供签名]
```

**在继续进行 GREEN 到签名之前:** 阅读 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 中的 `## Who's using this`。如果角色是非律师:

> 此步骤有法律后果(连署 NDA 约束公司)。你是否已与律师审查此内容?如果是,继续。如果不是,这是带给他们的简报:
>
> [生成 1 页摘要:对手方、NDA 方向(互惠/单向)、运行的剧本检查、剧本未涵盖的内容、如果按原样签署可能出什么问题,以及要问律师的三件事。]
>
> 如果你需要找到律师、事务律师、大律师或其他授权法律专业人士:联系你专业监管机构(美国的州律协、英格兰和威尔士的 SRA/Bar Standards Board、苏格兰/NI/爱尔兰/加拿大/澳大利亚的 Law Society,或你司法管辖区的同等机构)以获取推荐服务。

在没有明确是的情况下,不要越过此关卡。

### 黄——需要律师关注具体事项

一个或多个条款偏离剧本但不是绝对的交易破坏者,或者出现了剧本未解决的条款。单独浮现每个项目,以便审批人可以做出决定。

**输出:**

在前面加上 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` `## Outputs` 中的工作产品标题(根据用户角色而不同——见 `## Who's using this`)。

```markdown
[WORK-PRODUCT HEADER — per plugin config ## Outputs]

## NDA 分类: [对手方]

黄——标记供[来自 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 的审批人姓名]

### 执行摘要

- [一行可执行编辑,例如,"删除非招揽条款(第 6 节)"]
- [一行可执行编辑]

### 标记项目

**1. [问题]** —第 [X] 节
   什么: [一行]
   为什么标记: [一行——这触及哪个剧本立场,或"剧本对此保持沉默"]
   **法律风险:** [🔴/🟠/🟡/🟢] | **业务摩擦:** [🔴 阻塞交易 / 🟠 减慢交易 / 🟡 困惑客户 / 🟢 不可见]
   可能的解决方案: [接受 / 对 X 推回 / 取决于交易背景]

[重复每个标志]

### 其他所有内容

| 检查 | 状态 | 剧本参考 |
|---|---|---|
| [通过的剧本检查] | 通过 | [`~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 部分] |

**下一步:** 询问[审批人]关于标记项目,如果他们对项目没问题,则路由到签名。
```

### 红——停止,首先与法律交谈

NDA 触及剧本"永不接受"列表上的立场,或者协议结构与团队的标准姿态不兼容(例如,团队剧本要求互惠时的单向 NDA;剧本上限为有限期时的永久条款;管辖法律在"永不"列表上)。

**输出:**

在前面加上 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` `## Outputs` 中的工作产品标题(根据用户角色而不同——见 `## Who's using this`)。

```markdown
[WORK-PRODUCT HEADER — per plugin config ## Outputs]

## NDA 分类: [对手方]

红——不要提交,首先与法律交谈

### 执行摘要

- [一行可执行编辑,例如,"第 4 节——路由供法律审查"]
- [一行可执行编辑]

### 关键问题

**1. [问题]** —第 [X] 节
   > "[确切引用]"
   为什么这是个问题: [具体风险;引用其违反的剧本立场]
   **法律风险:** [🔴/🟠/🟡/🟢] | **业务摩擦:** [🔴 阻塞交易 / 🟠 减慢交易 / 🟡 困惑客户 / 🟢 不可见]
   建议响应: [使用我们的文件 | 用具体语言推回 | 走开]

**下一步:** 将此分类发送给[GC 或来自 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 的命名升级人]。不要发送到 [CLM 或审批工作流程]。不要告诉对手方我们会签署。
```

## 红线粒度

**以尽可能小的粒度编辑。** 红线是谈判产物,而非重写。整条替换发出信号"我们丢弃了你的起草"——这是激进的,它迫使对手方重新阅读整个条款,并且丢弃其起草中很好的部分。外科红线——删除一个词、插入一个短语、重组一个子句——发出"我们有具体要求"的信号,并且更快阅读、理解和接受。

默认为最小化实现剧本立场的编辑:
- 在短语之前替换**词**。(将"十二(12)"替换为"二十四(24)")
- 在句子之前替换**短语**。(将"由买方支付"替换为"由买方支付并应付")
- 在替换句子之前重组**子句**。(添加"(a)"和"(b)"以分割复合条件。)
- 在替换条款之前替换**句子**。
- 仅在对手方版本距你的立场太远以至于外科编辑比新草案更难阅读时替换**整个条款**——当你这样做时,在传输中说明:"我们替换了 §8.2 而不是标记它,因为更改很广泛。很高兴引导你了解差异。"

如有疑问,更小。收到外科红线的客户信任你仔细阅读。收到整批替换的客户怀疑你是否根本没读。

## 司法管辖区假设

此分类应用 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 中记录的管辖法律和限制性契约立场。法律规则(非招揽、非竞争、费用转移、选择法律)的可执行性因司法管辖区而有很大差异。如果 NDA 涉及团队配置姿态之外的司法管辖区,在输出中标记它并注意分类可能按书面转移。

## 输出规则

**复杂性过滤器:** 如果解决问题需要起草新语言、重组条款或插入实质性新条款——不要尝试。改为写:"第 [X] 节——路由供法律审查。"仅在执行摘要中包含简单、机械的行动(删除、删除、替换词或短语)。

**干净 NDA 规则:** 如果 NDA 通过所有检查且无标志,执行摘要应仅说:"根据剧本未识别红旗。按标准流程路由以供签名。"

不要为干净的 NDA 生成长报告。

## 详细检查参考

对于下面的每个检查,存储桶(GREEN/YELLOW/RED)由 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 确定。此 skill 列出要检查的*类别*;它不对阈值进行硬编码。

### 互惠性

NDA 是互惠还是单向?应用 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 中的团队立场。如果剧本未解决此上下文的单向 NDA,运行下面的单向问卷并将结果浮现给人类。

**单向 NDA 问卷**

当 NDA 是单方面的(一方披露,另一方仅接收)时,不要立即标记红旗或退出。问:

> 在某些情况下,单向 NDA 是合适的。在标记此之前,
> 让我问几个快速问题:
>
> 1. 在此关系中,你是唯一披露保密信息的一方吗?(即,另一方什么都不回分享)
> 2. 这是否用于有限的、特定的披露——例如,与将处理它的供应商分享你的技术,但不与我们分享他们的技术?
> 3. 这是否与 M&A、雇佣或投资相关?(如果是,停止——此 skill 仅用于商业 MNDAs。路由供法律审查。)

使用答案加上 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 立场来决定绿/黄/红。如果 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 未对此事实模式采取立场,标记黄色并将问卷答案浮现给审批人。

### 保密信息定义

检查范围(仅标记 vs. 全部披露)、标记要求和口头披露确认窗口。应用 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 中的团队立场。如果剧本对其中任何一个保持沉默,询问。

### 例外

NDA 中通常存在的五个例外:

1. 信息是或变成公开的(非通过违规)
2. 接收方已经拥有的信息
3. 接收方在不参考 CI 的情况下独立开发的信息
4. 从无限制的第三方收到的信息
5. 法律或法院命令要求披露的信息(在法律允许的情况下向披露方发出通知)

团队要求哪些例外以及多严格是剧本问题。检查 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 中的团队立场,关于所需例外、措辞的可接受变化,以及缺失一个时会发生什么。

### Residuals

Residuals 条款允许接收方使用保留在未辅助记忆中的信息。这是否可接受——以及在什么条件下(例如,狭窄的"未辅助记忆"措辞 vs. 涵盖注释或副本的更广泛范围)——是剧本问题。应用 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md`。如果剧本未解决 residuals,询问。

### 期限和存续

检查初始期限长度、保密义务的后期存续期,以及商业秘密是否以更长的保护被排除。应用 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 中的团队立场。如果剧本未涵盖其中之一,询问。

### 限制性契约

检查非招揽(员工、客户)、非竞争、排他性,以及对接收方可以与谁合作的任何限制。应用 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md`。如果剧本保持沉默,询问——限制性契约是司法管辖区敏感的,团队姿态很重要。

### 律师费

检查费用转移条款以及它们是互惠的、单边的还是胜诉方的。应用 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md`。

### 备份和归档例外

检查销毁/退还条款是否包括标准备份和归档保留系统的例外。应用 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 中的团队立场——一些团队要求此例外并将推动添加它;其他团队接受没有它的 NDA。如果剧本未解决此,询问。

### 管辖法律

根据 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` `## 剧本` → `管辖法律和地点`。

## 对手方背景

**BigCo NDA:** 财富 500 强对手通常不会谈判 NDA。校准:红旗是否真的是交易破坏者,还是"与我们的形式不同"?如果业务关系重要,决定是接受他们的文件——升级该决定,而非做出决定。

**初创公司 NDA:** 通常会接受我们的文件。如果他们的 NDA 有问题,最快的路径通常是"让我们使用我们的",而不是红线他们的。

## 集成:CLM

如果连接:
- 绿 → 提议在标准 NDA 工作流程中创建 CLM 记录
- 黄 → 提议创建它,并附上列出标记项目的注释
- 红 → 不创建记录;律师决定接下来发生什么

## 此 skill 不做什么

- 它不谈判。它分类。
- 它不起草 NDA。如果答案是"使用我们的文件",用户从 [CLM 或文档系统] 拉取我们的表格。
- 它不对黄项目做出决定。它将它们浮现给人类。
- 它不对任何 NDA 条款陈述立场。立场存在于 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 中。

## 结束行动

阅读 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` → `## NDA 分类偏好` → `closing_action`。

如果配置,在每个输出末尾逐字附加结束行动。示例配置:

```
closing_action: "将此分析的全文连同 NDA 副本发送给
legal@[yourcompany].com 以供签署前最终确认。"

closing_action: "使用标准 NDA 工作流程提交到 [CLM]。
法律将在路由以供签名前确认。"

closing_action: "将此输出和 NDA 转发给你的合同经理。"
```

如果 `closing_action` 在 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 中未配置,附加:
"通过你的标准审批流程路由最终 NDA。"

冷启动访谈问:"当有人完成 NDA 分类时,你希望他们对输出做什么?我会将其作为每个审查的常设指令添加。"

## 以下一步决策树结束

根据 CLAUDE.md `## Outputs` 以下一步决策树结束。根据此 skill 刚刚生成的自定义选项——五个默认分支(起草 X、升级、获取更多事实、观察等待、其他)是起点,而非锁定。树就是输出;律师选择。
