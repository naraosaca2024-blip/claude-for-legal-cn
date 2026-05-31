---
name: policy-redraft
description: 生成标记的政策重拟提案，以关闭 /regulatory-legal:gaps 或 /regulatory-legal:policy-diff 发现的差距。内部审查的初稿——不直接应用于批准的政策文档。当用户说"重拟政策"、"起草政策修复"、"标记政策"，或当 gap-surfacer 移交差距以供起草时使用。
argument-hint: "[GAP-ID 或差距描述]"
---

<!--
This file is a Chinese translation of the original by Anthropic PBC.
Original: https://github.com/anthropics/claude-for-legal
Licensed under Apache License 2.0
-->


# /policy-redraft

1. 加载 `~/.claude/plugins/config/claude-for-legal/regulatory-legal/CLAUDE.md` → 政策库索引 + 执业档案。
2. 使用以下工作流。
3. 收集输入：差距（来自 `/regulatory-legal:gaps` 输出或直接描述）、当前批准的政策文本、规则文本。
4. 验证规则是当前的（根据 policy-diff 规则状态检查）。如果无法验证，发出 `⚠️ RULE STATUS UNVERIFIED` 横幅。
5. 生成受影响政策章节的标记重拟——尽可能最小的编辑、携带 `[verify]` 标签、内联评论解释每个更改的原因。
6. 输出政策重拟备忘录。将其写入名为 `[policy-name]-proposed-redraft-[YYYY-MM-DD].md` 的新文件——永远不要写入源政策文档。
7. 不要关闭跟踪器中的差距。差距仅在重拟被应用并批准时关闭，这是政策所有者的行动。

---

> 此 skill 生成**提案**，而非编辑。它写入具有明确标记的草案文件名的新文件。它永远不写入源政策文档，也永远不关闭跟踪器中的差距——差距仅在重拟被政策所有者应用并批准时关闭。

## Matter context

**Matter context.** 检查实践级 CLAUDE.md 中的 `## 事项工作区`。如果 `Enabled` 是 `✗`（内部用户的默认值），跳过此段的其余部分——skills 使用实践级上下文，事项机制不可见。如果启用且没有活动事项，询问："这是哪个事项？运行 `/regulatory-legal:matter-workspace switch <slug>` 或说 `practice-level`。"加载活动事项的 `matter.md` 以获取事项特定的上下文和覆盖。将输出写入事项文件夹 `~/.claude/plugins/config/claude-for-legal/regulatory-legal/matters/<matter-slug>/`。除非 `跨事项上下文` 是 `on`，否则绝不读取另一个事项的文件。

---

## 目的

Gap-surfacer 发现差距。Policy-diff 命名需要更改的内容。此 skill 采取下一步并生成受影响政策章节的标记重拟——小、具体、标记——作为政策所有者审查的初稿。

## 硬性护栏——首先阅读这些

这些是承载规则。如果其中任何一条将被违反，停止并询问。

1. **这是提案，不是编辑。** 永远不要直接写入源政策文档。输出进入新文件 `[policy-name]-proposed-redraft-[YYYY-MM-DD].md`，或进入事项工作区。不是 `[policy-name].md`。
2. **永远不要关闭跟踪器中的差距。** 差距在重拟被应用并批准时关闭——那是政策所有者的行动，不是您的。如果用户说"既然你已重拟，现在关闭差距"，拒绝："我生成提案。当您审查、应用并批准更改时，差距关闭。完成后告诉我，我将更新跟踪器。"
3. **"为我应用"不在范围内。** 如果用户要求您将重拟应用到源政策："我不应用策略更改——那是审查和批准后政策所有者的行动。我生成提案。当它已审查和批准时，告诉我，我将更新差距跟踪器。"
4. **在重拟之前确认政策版本。** 如果用户给您文件，询问："这是批准的政策版本吗，是最新的吗？针对过时策略的重拟会产生分歧。"如果他们粘贴文本，相信但在审查者笔记中标记。
5. **尽可能最小的编辑。** 在句子之前删除一个词，在段落之前删除一个句子，在章节之前删除一个段落。仅触及受差距影响的章节。不要重新设计策略。
6. **携带 `[verify]` 标签。** 任何来自模型知识或未验证来源的生效日期、阈值、引用或要求都会在重拟本身中标记，而不仅仅是在备忘录中。

## 步骤 1：收集输入

需要三个输入。如果任何输入缺失，询问——不要推断。

### 1a. 差距

以下之一：
- 来自差距跟踪器的 `GAP-ID` — 从 `~/.claude/plugins/config/claude-for-legal/regulatory-legal/gap-tracker.yaml`（或事项级等效项）加载条目。
- 在用户消息中描述的差距——捕获要求、法规和受影响的政策。
- 从 `/regulatory-legal:policy-diff` 输出粘贴的差异摘要。

### 1b. 当前政策文本

以下之一：
- 文件路径——读取它，然后询问："这是批准的政策版本吗，是最新的吗？针对过时策略的重拟会产生分歧。"在审查者笔记中记录答案。
- 粘贴的文本——相信但在审查者笔记中标记："政策文本是直接粘贴的；我假设它是当前批准版本。在应用前确认。"
- 都不是——询问一个。不要从差距跟踪器或网络搜索猜测政策文本。

### 1c. 规则文本

以下之一：
- 差异输出（已有规则提取和标记）。
- 获取的法规——用来源标签注明来源。
- 用户粘贴的规则文本——标记 `[user provided]`。

如果规则文本部分或模棱两可，应用 CLAUDE.md 中的 **no silent supplement** 规则：向用户提供选项（粘贴全文、指向主要来源、带验证标签的网络搜索、或停止），并等待。

## 步骤 2：验证规则是当前的

使用与 `policy-diff` 相同的规则状态检查模式。红旗表明规则可能未生效：

- 适用性/合规日期已超过 30 天以上，没有确认它没有被延迟。
- 规则超过 12 个月。
- 规则是政治上有争议的最终规则（主要规则制定经常受到挑战）。

当您看到红旗时，检查（通过研究 MCP、如果启用的网络搜索或 Federal Register docket）：延迟、停留、禁令、废除提议、撤销或修正。如果您可以验证规则生效，继续。如果您无法验证：

> `⚠️ RULE STATUS UNVERIFIED — 我无法确认此规则当前生效。最终规则在发布后经常被停留、禁令、延迟或废除。在确认规则状态之前不要应用此重拟，在 Federal Register docket 或与外部律师确认。`

在工作产品标题上方发出该横幅。将重拟中的每个生效/合规日期标记为 `[effective date per published rule — status unverified]`。

## 步骤 3：生成重拟

受影响政策章节的标记版本。

### Redline 粒度——尽可能最小的编辑

- 在句子之前删除一个词。
- 在段落之前删除一个句子。
- 在章节之前删除一个段落。
- 仅触及受差距影响的章节。不要重新设计整个政策。

### 约定

- 删除的文本：`~~struck text~~`
- 插入的文本：**inserted text**
- 每个更改携带内联评论解释原因——规则、引用、正在关闭的差距：

  > `[Change: per COPPA 2025 amendments, 16 CFR 312.2 (生效于 2026 年 4 月 22 日) [verify]，将生物标识符添加到 PII 定义]`

- 任何来自模型知识或未验证来源的生效日期、阈值、引用或要求都会获得内联 `[verify]` 标签——不仅仅在变更摘要中。
- 从差异携带源标签：`[Federal Register]`、`[web search — verify]`、`[model knowledge — verify]`、`[user provided]`。在从差异移动到重拟时不要剥离它们。

### 范围纪律

如果政策的一部分不受差距影响，请勿干涉。触及差距之外章节的重拟看起来像 AI 对未被要求发表意见的事情发表了意见，使审查更困难。

如果您在重拟时看到第二个差距——显然与规则不一致但不在原始差距中的条款——不要静默修复。在审查者笔记中标记："在为 [GAP-ID] 重拟时，我注意到 [其他条款] 似乎与 [要求] 有相关问题。未包含在此重拟中。考虑后续差距。"

## 步骤 4：输出——政策重拟备忘录

```markdown
[工作产品标题 — 根据插件配置 ## 输出 — 因角色而异；参见 `## 谁在使用此`]

> **⚠️ Reviewer note**
> - **Sources:** [研究连接器：CourtListener ✓ verified | 未连接——来自训练知识的引用，在依赖前验证]
> - **Read:** [审查的策略章节；未阅读的内容]
> - **Flagged for your judgment:** [内联标记 `[review]` 的 N 个项目 | 无]
> - **Currency:** [规则状态根据 [来源], [日期] 验证 | 未验证——参见上面的横幅]
> - **Before relying:** 确认这是当前批准的政策版本；验证规则状态和生效日期；获得政策所有者的审查；遵循您的政策更改批准流程；仅在应用和批准时更新差距跟踪器。

## Policy Redraft: [Policy name]

**Gap:** [GAP-ID 或简短描述]
**Regulation:** [名称、引用、生效日期]
**Policy:** [名称、最后更新日期]
**Status:** PROPOSAL — 尚未审查或批准

### Conclusion

[一句话：差距是什么。一句话：重拟做什么。一句话：需要审查。]

### Marked-up policy section(s)

[带有内联 `[Change: ...]` 评论的 redline 文本。仅受影响的章节。]

### Change summary

| # | Provision | Current | Proposed | Why | Verify |
|---|---|---|---|---|---|
| 1 | §2.1 PII definition | "…names, addresses, SSNs…" | "…names, addresses, SSNs, biometric identifiers…" | COPPA 2025 amendments expand PII to cover biometrics | [Federal Register] |
| 2 | §4.3 Retention period | "30 days" | "14 days" | New rule imposes 14-day cap | `[verify — model knowledge]` |

### Before applying — checklist

- [ ] 确认这是当前批准的政策版本正在重拟。
- [ ] 验证规则状态和生效日期（Federal Register docket 或外部律师）。
- [ ] 获得政策所有者的审查。
- [ ] 遵循您的政策更改批准流程。
- [ ] 在应用和批准时更新差距跟踪器——而非之前。

---

**What next? Pick one and I'll help you build it out:**

1. **Apply and get sign-off** — 您审查，分发给政策所有者，通过您的批准流程。批准后，告诉我，我将标记差距关闭。
2. **Get more info on [X]** — 如果特定更改需要更多基础（验证引用、检查阈值、解决司法管辖区问题），告诉我哪一个，我将深入。
3. **Escalate to [owner / GC]** — 如果重拟提出超出政策所有者权限的事情，我将起草简短升级，包括事实、建议更改和所需决定。
4. **Watch and wait** — 如果规则状态不确定或政策所有者不可用，我将向差距跟踪器添加重新访问笔记。
5. **Something else** — 告诉我您会怎么做。
```

## 文件名

输出文件名明确表明它是草案。使用：

`[policy-name]-proposed-redraft-[YYYY-MM-DD].md`

不是 `[policy-name].md`。不是 `[policy-name]-v2.md`。"proposed-redraft"一词和日期是承载的——它们防止草案被误认为是当前版本。

如果有活动事项则写入事项工作区；否则写入当前工作目录或用户命名的位置。不要写入政策库源目录。

## 配置依赖的后备方案

此 skill 从 `~/.claude/plugins/config/claude-for-legal/regulatory-legal/CLAUDE.md` 读取政策库索引和所有者。当它需要的值为空或仍然是 `[PLACEHOLDER]` 时：

- **缺少政策所有者：** 仍然生成重拟。在审查者笔记中注明："在 `## Policy library` 中没有为 [政策] 设置政策所有者。使用 `/regulatory-legal:cold-start-interview --redo` 分配一个，以便批准路径可路由。"
- **政策库为空且差距未命名特定政策：** 停止并询问："我需要当前政策文本来重拟。粘贴受影响政策的文本，或指向文件。"

当值已填充时，不说明配置。

## 与其他 skills 的交互

- **上游输入**来自 `policy-diff`（每要求差距分析）和 `gap-surfacer`（跟踪器）。携带其源标签和 `[verify]` 标志。
- **差距跟踪器状态：** 此 skill 不更改跟踪器。它不标记差距关闭、不标记进行中、不触及 `notified`。如果您需要重拟存在的纸质记录，政策所有者或用户可以在重拟应用和批准时更新差距条目并附解决方案笔记——参见 `/regulatory-legal:gaps --close`。
- **严重性下限：** 如果上游差距是 🔴 或 🟠，备忘录的底线具有该严重性。静默降级是审查律师无法看到的矛盾。参见 CLAUDE.md `## 跨技能严重性下限`。

## 用下一步决策树结束

包含在上面的输出模板中。根据重拟实际生成的内容自定义选项——如果规则状态未验证，选项 2（获取更多信息）上移；如果未设置政策所有者，选项 3（升级）变得具体。

## 此 skill 不做什么

- 将重拟应用到源政策。那是政策所有者的行动。
- 关闭跟踪器中的差距。差距在重拟应用和批准时关闭。
- 重写整个政策。尽可能最小的编辑来关闭差距。
- 生成多政策重拟。一个差距、一个政策、一个备忘录。用于多政策扩展的 `:package` 命令是未来的 skill。
- 生成"应用"工作流。带有批准关卡的 `:apply` 命令是未来的 skill。
