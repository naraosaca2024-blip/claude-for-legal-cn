---
name: reg-feed-watcher
description: 立即检查监管订阅源并报告自上次检查以来的新内容，根据您的重要性阈值进行过滤。当用户说"检查订阅源"、"有什么新内容"、"监管更新"、从计划代理运行时，或手动粘贴监管发展进行分类和差异时使用。
argument-hint: "[可选：--since DATE]"
---

<!--
This file is a Chinese translation of the original by Anthropic PBC.
Original: https://github.com/anthropics/claude-for-legal
Licensed under Apache License 2.0
-->


# /reg-feed-watcher

1. 加载 `~/.claude/plugins/config/claude-for-legal/regulatory-legal/CLAUDE.md` → 监视列表、重要性阈值、订阅源配置。
2. 使用以下工作流。
3. 拉取每个订阅源。按重要性过滤。
4. 输出：有什么新内容，按重要性层级分类。

---

## 目的

拉取订阅源。按重要性过滤。输出剩余内容。过滤器是价值——未过滤的订阅源只是噪音。

## 加载上下文

`~/.claude/plugins/config/claude-for-legal/regulatory-legal/CLAUDE.md` → 监视列表、重要性阈值、订阅源配置、摘要输出路径（如果已设置）。

`references/source-catalog.md`（在此 skill 目录中）→ 跨美国联邦、美国州、欧盟/英国、国际以及二级/聚合商类别的 RSS/JSON/HTML 源的精选目录。在配置新源时或用户的监视列表存在覆盖缺口时使用（参见步骤 0）。

## 工作流

### 步骤 0：覆盖检查（拉取之前）

在运行拉取之前，比较 CLAUDE.md 中的监视列表 + 订阅源配置与 `references/source-catalog.md`：

- 根据他们的监视列表，用户关心哪些类别（美国联邦 / 美国州 / 欧盟-英国 / 国际）？
- 这些类别中有哪些配置了零个或很少的源？

如果存在明显缺口——例如，用户在监视列表中监视"欧盟监管机构"，但在订阅源中仅配置了 `edpb.europa.eu`，缺少 ICO、CNIL、DPC Ireland——在摘要顶部一次性提出：

> **覆盖缺口：** 您的监视列表包括 [类别]，但仅配置了 [N] 个订阅源。源目录在此类别中列出了 [X] 个选项（例如，[前 2-3 个名称]）。要我建议添加吗？运行 `/regulatory-legal:cold-start-interview --redo` 进行更新，或直接编辑 `~/.claude/plugins/config/claude-for-legal/regulatory-legal/CLAUDE.md`。

不要反复唠叨相同的缺口——如果用户明确表示"暂时跳过州检察长"，请尊重。在 CLAUDE.md 中记录状态以便保持。

### 步骤 1：拉取

从所有配置的订阅源层级拉取。每个安装都有第 1 层级。第 2 和第 3 层级是附加的——如果配置了则使用，否则跳过。

**第 1 层级 — 免费订阅源（始终活跃）**

对于监视列表中的每个监管机构：

- **Federal Register API** (`https://www.federalregister.gov/api/v1/documents`)
  — 按机构 slug、日期范围（自上次检查以来）、文档类型查询。返回
  结构化数据：文档类型、标题、摘要、生效日期、评论
  截止日期（对于 NPRM）和引用。覆盖所有美国联邦机构。
- **直接监管机构 RSS** — 获取并解析 ~/.claude/plugins/config/claude-for-legal/regulatory-legal/CLAUDE.md 订阅源配置中的任何 RSS URL
  （SEC、FTC、CFPB、州机构、欧盟 DPA 等）。

常见监视列表监管机构的机构 slug 参考：
| 监管机构 | API slug |
|---|---|
| FTC | federal-trade-commission |
| SEC | securities-and-exchange-commission |
| CFPB | consumer-financial-protection-bureau |
| CPPA (CA) | 仅 RSS — cppa.ca.gov/feed |
| DOL | labor-department |
| HHS | health-and-human-services-department |
| FCC | federal-communications-commission |

对于不在此列表中的任何监管机构：检查 federalregister.gov/agencies 以获取
正确的 slug，或回退到直接 RSS。

**第 2 层级 — 付费订阅源（如果配置）**

- **付费监管订阅源 MCP：** 查询自上次检查日期以来的更新，
  过滤到监视列表监管机构。
- **CourtListener MCP：** 相同。

跨层级去重——同一文档可能出现在多个源中。
优先为丰富输出使用最丰富的源。

**无静默补充。** 如果订阅源拉取为监视列表中的监管机构返回很少或没有结果，报告找到的内容并停止。不要在未询问的情况下从网络搜索或模型知识填补缺口。说："订阅源检查从 [命中的监管机构] 返回了 [N] 项。对于 [监管机构 / 主题]，覆盖似乎稀薄。选项：(1) 扩大日期窗口，(2) 尝试不同的订阅源或 MCP，(3) 搜索网络——结果将标记 `[network search — verify]`，在依赖之前应根据发行机构的网站进行检查，或 (4) 停在这里。您想要哪个？"律师决定是否接受较低置信度的来源；Claude 不为他们决定。

**来源归属。** 为每个引用和监管项目标记其来源：`[Federal Register]`、`[<监管机构> RSS]`、`[CourtListener]` 或通过连接器检索的项目的特定 MCP 工具名称；`[web search — verify]` 用于来自网络搜索的项目；`[model knowledge — verify]` 用于从模型训练数据中浮现的项目；`[user provided]` 用于手动粘贴的项目。带有 `verify` 标签的项目比工具检索的项目具有更高的伪造风险，应首先检查。永远不要剥离或折叠标签——它们是用户了解要验证哪些引用的最快信号。

**次要来源。** 一些目录条目（IAPP、FPF、Hogan Lovells、Covington、Lexology、JD Supra、Artificial Lawyer、LawSites 和类似评论员/聚合商）报告主要监管行动，但不是主要来源。将从这些订阅源提取的任何项目标记为 `[secondary source]` 以及订阅源名称标签——例如，`[IAPP Daily Dashboard] [secondary source]`。在摘要中，当次要来源项目描述监管机构行动时，添加注释："→ 追溯到主要：[如果已知，链接到监管机构站点，否则'在依赖之前在 <监管机构>.gov 上查找']。不要仅凭次要来源项目的强度将其归类为"始终重要"——在找到主要来源之前将其降低一个层级。

**第 3 层级 — 手动输入**

如果用户粘贴了监管文本或摘要而不是从
计划的订阅源检查调用：将粘贴的内容视为单个项目，跳至
步骤 2 进行分类，并记录来源为"手动输入"。不需要订阅源拉取。
无论订阅状态如何，此路径都有效。

拉取后记录检查时间戳。下一次计划运行从此处
向前拉取。

### 步骤 2：分类

每个项目根据 `~/.claude/plugins/config/claude-for-legal/regulatory-legal/CLAUDE.md` 获得重要性层级：

| 项目类型 | 与阈值匹配 |
|---|---|
| 最终规则 | 通常"始终重要" |
| 拟议规则 / NPRM | 通常"值得审查"——并始终记录评论截止日期 |
| ANPR（拟议规则制定提前通知） | 对于**策略**值得审查，而不是合规——尚未施加要求，但发出方向信号并带有真正的评论截止日期。记录评论截止日期。仅作为预先定位分析路由到 `/regulatory-legal:policy-diff`，而不是差距关闭差异。 |
| RFI（信息请求） | 与 ANPR 相同——规则前，无合规义务，但评论截止日期是真实的，方向发出信号是价值。 |
| 执法行动 | 行业匹配 → 重要；相关实践匹配 → 值得审查；两者都不 → 仅供参考 或 跳过 |
| 指导 | 值得审查 |
| 演讲 / 博客 / 声明 | 仅供参考 或 根据阈值跳过 |
| 和解 | 取决于——新颖理论或大数字 → 值得审查；常规 → 跳过 |

**ANPR / RFI 处理 — 具体说明。** 规则前项目在一个重要方面与 NPRM 不同：它们不改变法律，但它们确实带有评论截止日期并且它们发出监管机构的方向信号。将它们视为一个单独的分支：

- **不要**将 ANPR / RFI 归类为"始终重要"——在规则发布之前合规影响为零。
- **如果通知中的任何问题领域触及监视列表的始终重要类别（例如，金融科技监视列表中关于开放银行 ANPR），则**归类为值得审查。
- **将评论截止日期记录到** `~/.claude/plugins/config/claude-for-legal/regulatory-legal/comment-tracker.yaml`，并带有 `item_type: ANPR` 或 `item_type: RFI`，以便下游跟踪器可以将其与合规差距区分开来。
- **在摘要条目中包含一行**，明确指出："规则前。评论截止日期 [日期]。仅作为预先定位分析路由到 `/regulatory-legal:policy-diff`（尚无合规差距）。"这引导 policy-diff skill 使用其压缩的预先定位分支，而不是完整的差距关闭差异。
- **路由到评论跟踪器，而不是差距跟踪器。** 评论决策项目不是合规差距；它们属于评论跟踪器，`gap-surfacer` 使用 `comment-decision` `gap_type`（或拒绝摄取，如果团队单独路由这些）。

**NPRM 评论截止日期处理：**

对于以任何高于"跳过"层级分类的每个 NPRM：
- 提取评论截止日期（Federal Register API 将此作为结构化数据返回）
- 如果在 ~/.claude/plugins/config/claude-for-legal/regulatory-legal/CLAUDE.md 中启用了评论跟踪：附加到 `~/.claude/plugins/config/claude-for-legal/regulatory-legal/comment-tracker.yaml`
  ，状态为"undecided"，以及来自 ~/.claude/plugins/config/claude-for-legal/regulatory-legal/CLAUDE.md 的默认评论决策所有者
- 在输出条目中包含评论截止日期

### 步骤 3：丰富

对于每个高于仅供参考 层级的项目：

- 一行摘要（改变了什么）
- 为什么这里可能重要（相关性钩子——"这是关于 [您所做的实践]"）
- 指向源的链接
- 生效日期或评论截止日期（如适用）

不要单独总结仅供参考 项目——只计算它们。

## 输出

摘要默认进入聊天。**同时将其写入可共享文件**，每当输出包含一个或多个高于仅供参考 的项目时，除非用户的 CLAUDE.md 明确设置 `摘要输出 → 仅聊天`。

**文件输出行为：**

1. 在 `~/.claude/plugins/config/claude-for-legal/regulatory-legal/CLAUDE.md` 中查找 `摘要输出路径`。如果已设置，则写入此处。如果未设置的默认值：`~/regulatory-legal-digests/reg-digest-YYYY-MM-DD.md`。
2. 如需要则创建父目录。
3. 将完整摘要写入为 Markdown（与聊天输出内容相同，包括工作产品标题、源标签和 verify-citations 页脚）。
4. 如果该路径今天已存在文件，则附加带有时间戳子标题的新部分而不是覆盖——同一天可能会看到多次运行（早间摘要、临时检查）。
5. 写入后，告诉用户："摘要已写入 `<path>`。按原样共享，或使用 Pandoc 转换为 .docx：`pandoc <path> -o <path>.docx`。"
6. 如果写入失败（权限、用户未授权创建的缺失目录、磁盘），回退到仅聊天输出并说明——不要静默删除文件请求。

磁盘上的格式与聊天格式完全匹配（下方）。Markdown 在 GitHub、Notion、Obsidian、Google Docs（通过"作为 Markdown 导入"或 Pandoc）以及大多数电子邮件客户端中呈现良好。

```markdown
[工作产品标题 — 根据插件配置 ## 输出 — 因角色而异；参见 `## 谁在使用此`]

## 监管订阅源检查 — [日期]

**期间：** [上次检查] 到 [现在]
**检查的订阅源：** [活动层级列表 — 例如，"Federal Register API、FTC RSS、TR"]
**找到的项目：** 总共 [N]

### 结论

[N 个差距需要在 [日期] 前采取行动 — 前 3 个：X、Y、Z]

### 🔴 始终重要

**[监管机构] — [标题]**
[一行摘要]。[相关性钩子]。生效日期 [日期]。
[链接]
→ 建议：对 [可能受影响的策略] 运行 policy-diff

[每个项目重复]

### 🟡 值得审查

**[监管机构] — [标题]**
[一行]。[相关性]。[截止日期（如有）]。
[链接]

[NPRM：如果启用评论跟踪，包含"💬 评论截止日期：[日期] — 待决策"]

[重复]

### 📝 仅供参考

[N] 个项目 — [可展开的标题 + 链接列表，无摘要]

---

**上次检查更新至：** [时间戳]
**评论跟踪器：** [N] 个 NPRM 有开放的评论决策 — 运行 /regulatory-legal:comments 进行审查

---

**在依赖之前验证引用。** 此处的监管引用是 AI 生成的，尚未与主要来源核对。在采取任何规则、指导或执法行动之前，根据 Westlaw、您事务所的研究平台或发行机构的网站进行确认——检查准确性、生效日期和当前状态。AI 生成的监管引用有时会被伪造、误引用或过时。每个项目上的源标签（例如，`[Federal Register]`、`[web search — verify]`）显示引用的来源；带有 `verify` 的标签具有更高的伪造风险，应首先检查。
```

## 配置依赖的后备方案

此 skill 从 `~/.claude/plugins/config/claude-for-legal/regulatory-legal/CLAUDE.md` 读取监视列表、重要性阈值和订阅源配置。当所需值仍然是 `[PLACEHOLDER]` 或为空时，在输出中具体说明——具体来说，而不是笼统说明：

- **监视列表为空：** 停止并说"您配置中的监视列表为空。我不知道要监视哪些监管机构，无法拉取订阅源。运行 `/regulatory-legal:cold-start-interview --redo` 或编辑 `~/.claude/plugins/config/claude-for-legal/regulatory-legal/CLAUDE.md` 并添加至少一个监管机构。"
- **重要性阈值为空：** 回退到默认层级并附加："此输出使用了默认重要性层级，因为您的配置没有设置自定义阈值。使用 `/regulatory-legal:cold-start-interview --redo` 或通过编辑 `~/.claude/plugins/config/claude-for-legal/regulatory-legal/CLAUDE.md` 进行调整。"
- **订阅源配置为空：** 仅运行 Federal Register API 并附加："此输出仅使用了免费的 Federal Register API，因为您的配置没有列出直接 RSS 或付费订阅源。使用 `/regulatory-legal:cold-start-interview --redo` 或通过编辑 `~/.claude/plugins/config/claude-for-legal/regulatory-legal/CLAUDE.md` 添加订阅源。"

当相关值已填充时，不说明配置。

如果没有任何高于仅供参考 的项目："一切正常。[N] 个仅供参考 项目，无需注意。"

## 移交

- **到 policy-diff：** 任何具有可能策略影响的"始终重要"项目 → 提议运行差异。
- **到 gap-surfacer：** 如果差异发现差距 → 已跟踪。
- **到 comment-tracker：** 任何分类为高于"跳过"的 NPRM → 如果启用跟踪，评论截止日期自动记录。

## 用下一步决策树结束

根据 CLAUDE.md `## 输出` 以下一步决策树结束。根据此 skill 刚刚生成的内容自定义选项——五个默认分支（起草 X、升级、获取更多事实、观察等待、其他事情）是起点，而不是锁定。树就是输出；律师选择。

## 此 skill 不做什么

- 完整阅读每个项目。它进行分类和丰富；深度阅读适用于
  通过过滤的项目。
- 更改重要性阈值。如果过滤器错误，编辑 ~/.claude/plugins/config/claude-for-legal/regulatory-legal/CLAUDE.md。
- 需要 TR 或 CourtListener。免费订阅源是基准；付费订阅源增加深度。
