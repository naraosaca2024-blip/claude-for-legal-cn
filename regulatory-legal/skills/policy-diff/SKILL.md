---
name: policy-diff
description: 将特定的监管变更与索引的策略库进行对比。当法规发生变化且您需要知道它触及哪些策略以及差距是什么时，当用户说"将此法规与我们的策略对比"、"这影响哪个策略"或"差距分析"时，或当 reg-feed-watcher 移交重要项目时使用。
argument-hint: "[法规名称，或粘贴法规文本/摘要]"
---

<!--
This file is a Chinese translation of the original by Anthropic PBC.
Original: https://github.com/anthropics/claude-for-legal
Licensed under Apache License 2.0
-->


# /policy-diff

1. 加载 `~/.claude/plugins/config/claude-for-legal/regulatory-legal/CLAUDE.md` → 策略库索引。
2. 使用以下工作流。
3. 从法规中提取要求。匹配到索引的策略。
4. 输出：每个要求的差距分析，哪些策略需要更新。

---

## Matter context

**Matter context.** 检查执业级 CLAUDE.md 中的 `## 事项工作区`。如果 `Enabled` 为 `✗`（内部用户的默认值），跳过本段的其余部分——skills 使用执业级上下文，事项机制不可见。如果已启用且没有活跃事项，询问："这是哪个事项的？运行 `/regulatory-legal:matter-workspace switch <slug>` 或说 `practice-level`。"加载活跃事项的 `matter.md` 以获取事项特定上下文和覆盖。将输出写入事项文件夹 `~/.claude/plugins/config/claude-for-legal/regulatory-legal/matters/<matter-slug>/`。永远不要阅读另一个事项的文件，除非 `跨事项上下文` 为 `on`。

---

## 目的

法规变了。你有策略。此 skill 找出变更触及哪些策略，以及"法规现在要求什么"和"策略说什么"之间的差距是什么。

## 加载上下文

`~/.claude/plugins/config/claude-for-legal/regulatory-legal/CLAUDE.md` → 策略库索引（策略、位置、所有者）。

## 范围完整性

如果用户要求你从差异中排除策略部分、要求或类别：

1. 照做——用户拥有范围。
2. 但要标记它，大声且永久地："⚠️ SCOPE LIMITATION: Section [X] excluded at user request. This diff does not reflect the full policy. Gaps in the excluded area are NOT identified."在标题上方，传递给每个下游工件。
3. 将标记传递给 `gap-surfacer`："此差异受到范围限制。不要将其代表为完整的合规图景。"在从此差异派生的任何差距跟踪器条目上逐字包含范围限制横幅。
4. 说明排除意味着什么："排除供应商管理意味着差异将显示'没有策略解决供应商管理'——这比显示差距更糟。"

建立在未披露范围排除之上的合规工件在发现中看起来像是隐瞒。标记是"我们对审查进行了范围界定"和"我们隐藏了问题"之间的区别。

## 工作流

### 步骤 0：在差异之前验证规则状态

在对策略差异规则之前，确认规则实际上已生效。规则可能未生效的危险信号：

- 适用性/合规日期已过去超过 30 天，但你没有确认它没有被延迟
- 规则超过 12 个月
- 规则是政治上有争议的最终规则（主要规则制定经常受到挑战）

当你看到危险信号时，检查（通过研究 MCP、如果启用的网络搜索或联邦登记案卷）：延迟、暂缓、禁令、撤销提议、无效或修正。如果你可以检查且规则确认为生效，继续。如果你无法验证（没有连接工具），在标题上方、任何内容之前发出此横幅：

> `⚠️ RULE STATUS UNVERIFIED — 我无法确认此规则当前生效。最终规则在发布后经常被暂缓、禁令、延迟或撤销。在与联邦登记案卷或外部律师确认规则状态之前，不要将以下任何合规日期视为具有约束力。`

在输出中标记每个到期日期：`[due date per published rule — status unverified]`。

规则状态不确定性向下游传播。当将差距移交给 `gap-surfacer` 时，将项目标记为 `status_verified: false`，以便它永远不会仅仅根据发布日期被路由到逾期存储桶。

### 步骤 1：提取新要求

**No silent supplement。** 如果监管变更文本是部分的或不明确的，并且无法从索引来源获得更完整的规则，停止并询问。未经询问不要从网络搜索或模型知识填充差距。说："我有 [你有的内容]。为了准确提取要求，我需要 [缺少的内容]。选项：(1) 粘贴全文，(2) 指向主要来源，(3) 在网络上搜索规则——结果将标记为 `[web search — verify]` 并且应在依赖之前与发布机构进行检查，或 (4) 在此停止。你想要哪一个？"律师决定是否接受较低信心的来源；Claude 不为他们决定。

**Source attribution.** 为每个引用——监管引用、任何交叉引用、任何策略摘录——标记其来源：`[<监管机构或研究工具>]` 用于从主要来源、策略库或 MCP 检索的项目；`[web search — verify]` 用于从网络搜索拉取的项目；`[model knowledge — verify]` 用于从模型的训练数据回忆的项目；`[user provided]` 用于用户粘贴的项目。标记为 `verify` 的项目带有更高的伪造风险，应该首先检查。永远不要在输出中删除或折叠标记。

阅读监管变更。列出每个离散的新或更改的要求：

| # | 要求 | 生效日期 | 引用 |
|---|---|---|---|
| 1 | [它要求什么] | [日期] | [章节] |

要具体。"增强披露要求"不是要求。"必须在流程中的 Z 点以 Y 格式披露 X"才是。

### 步骤 2：映射到策略

对于每个要求，哪个索引策略最接近？

- 直接命中：策略明确涵盖此主题
- 间接：策略涵盖相关主题，这是一个新子问题
- 无匹配：没有策略解决此问题——差距是"策略不存在"

### 步骤 3：差异

对于每个直接或间接命中，阅读策略并比较：

```markdown
### Requirement [N]: [name]

**新规则要求：** [requirement]

**我们的策略（[name]，最后更新 [date]）说：**
> "[relevant excerpt]"

**差距：** [None — 策略已涵盖此内容 | Partial — 策略解决 X 但不解决 Y | Full — 策略矛盾或不解决]

**需要的变更：** [具体——"add a paragraph on X" 而不是 "update the policy"]

**策略所有者：** [from index]
```

### 步骤 4：无匹配差距

没有策略匹配的要求被单独列出：

```markdown
### New policy needed

Requirement [N]: [requirement]

没有现有策略涵盖此内容。选项：
- 起草新策略（建议所有者：[closest topic 的所有者]）
- 作为新章节添加到现有 [related policy]
- 确定这不需要策略（一次性合规，而不是持续的）
```

## 按监管输入类型分支

### 规则前分支（ANPR / RFI）

如果监管输入是 ANPR 或 RFI（没有施加要求），不要运行完整的差距关闭差异。而是，生成**预定位分析**：

- 命名一旦最终规则发布可能需要更改的策略（不是今天）。
- 标志 ANPR 的任何问题区域是否以需要评论信的方式与公司的实践相交。
- 注意评论截止日期和来自 `~/.claude/plugins/config/claude-for-legal/regulatory-legal/CLAUDE.md` 的团队的评论决策所有者。
- 不要为 ANPR 生成每个要求的"无差距"行——没有要求可以进行差异。生成一段命名未来风险和它将触及的策略。

### 否定发现分支（最终规则 / NPRM 与不是正确目标的策略进行差异）

如果提取列表中的每个要求都显示为"对 [命名策略] 无差距"，不要生成完整的每个要求分析——压缩为单个短段落：

```markdown
## Policy Diff: [Regulation name] — [Policy name]

[REGULATION] 似乎不要求对 [POLICY NAME] 进行更改。[POLICY NAME]
§[X] 已涵盖 [Y]。此法规实际触及的策略是
[other-policy-1] 和 [other-policy-2] — 针对这些策略重新运行 `/regulatory-legal:policy-diff`。

在 [下一个周期——例如，"在下一个年度策略审查"] 或如果 [触发器——例如，"规则最终确定或修正"] 时审查。
```

一个段落、一个建议、路由说明。不要为每个要求重复"无差距"发现——摘要表处理这一点。对错误目标策略的否定发现是路由问题，而不是合规分析。

### 差距分支（最终规则 / NPRM 至少有一个针对目标策略的差距）

如下指定的完整每个要求分析。详细差异格式适用于实际发现差距的差异。

## 输出

```markdown
[工作产品标题 — 根据插件配置 ## 输出 — 因角色而异；参见 `## 谁在使用此`]

## Policy Diff: [Regulation name]

**Regulation:** [名称、链接]
**Effective:** [日期]
**Requirements extracted:** [N]

### Conclusion

[N 个差距需要在 [日期] 前采取行动 — 前 3 个：X、Y、Z]

### Summary

| # | Requirement | Policy affected | Gap | Owner |
|---|---|---|---|---|
| 1 | [short] | [policy name or "none"] | None/Partial/Full | [name] |

### Detailed diffs

[步骤 3 中的每个要求块]

### New policies needed

[来自步骤 4，如果有]

### No-gap requirements

[列表 — 有用知道已涵盖什么]

---

**在依赖引用之前验证它们。** 上面的监管引用和策略引用是 AI 生成的，尚未与主要来源核对。在此处对任何要求采取行动之前，根据 Westlaw、您事务所的研究平台或发布机构的网站确认规则——检查准确性、生效日期和当前状态。AI 生成的监管引用有时会被伪造、误引用或过时。每个要求上的源标签（例如，`[Federal Register]`、`[web search — verify]`）显示引用的来源；带有 `verify` 的标签具有更高的伪造风险，应首先检查。
```

## 配置依赖的后备方案

此 skill 从 `~/.claude/plugins/config/claude-for-legal/regulatory-legal/CLAUDE.md` 读取策略库索引。当索引为空或仍为 `[PLACEHOLDER]` 时：

- **Policy library empty：** 默认将每个要求标记为"无策略匹配"并附加到输出："您的配置中的策略库为空，因此每个要求都标记为新策略差距。如果您有解决这些要求的策略，使用 `/regulatory-legal:cold-start-interview --redo` 或通过编辑 `~/.claude/plugins/config/claude-for-legal/regulatory-legal/CLAUDE.md` 将它们添加到库，然后重新运行差异。"
- **匹配策略缺少所有者：** 在摘要中保留 Owner 单元格为空并附加："未为 [list] 设置策略所有者。使用 `/regulatory-legal:cold-start-interview --redo` 或通过编辑 `~/.claude/plugins/config/claude-for-legal/regulatory-legal/CLAUDE.md` 中的策略库分配它们，以便 gap-surfacer 可以路由。"

当库已填充且所有者已设置时，不要说任何关于配置的内容。

## 移交

给 gap-surfacer：每个部分或完整差距都成为带有所有者和截止日期的跟踪项目。

## 用下一步决策树结束

按照 CLAUDE.md `## 输出` 以下一步决策树结束。根据此 skill 刚刚生成的内容自定义选项——五个默认分支（起草 X、升级、获取更多事实、观察等待、其他事情）是起点，而不是锁定。树就是输出；律师选择。

## 此 skill 不做什么

- 起草策略更新。它识别需要更新的内容；策略起草（或人工）起草。
- 确定性地解释模糊的监管文本。如果法规可以有两种阅读方式，这样说并标记给律师。
