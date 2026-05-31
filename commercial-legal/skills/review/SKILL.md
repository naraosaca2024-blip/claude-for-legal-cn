---
name: review
description: >
  根据您的剧本审查供应商协议、NDA 或 SaaS 订阅。
  从标题中识别协议结构，路由到正确的审查 skill
  （vendor-agreement-review、nda-review、saas-msa-review），并将输出
  集成到单个备忘录中。当用户说"review this contract"、"check this
  MSA"、"is this NDA okay"、"look at this SaaS agreement"或附加入站
  协议进行审查时使用。
argument-hint: '[file path | Drive link | [CLM ID] | paste text]'
---

<!--
This file is a Chinese translation of the original by Anthropic PBC.
Original: https://github.com/anthropics/claude-for-legal
Licensed under Apache License 2.0
-->


# /review

根据 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 中的剧本审查入站协议。从标题中识别协议结构，选择适当的 skill，并且——如果启用了 confirm_routing——在继续之前与用户确认。

## 说明

1. **加载 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md`。** 如果存在占位符，停止并提示："Run `/commercial-legal:cold-start-interview` first — I need to learn your playbook before I can review against it."

   同时读取 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` → `## Review preferences` → `confirm_routing`。如果字段缺失，将其视为 `true`。

2. **获取协议：** 从文件路径、Drive 链接、[CLM ID] 或粘贴的文本。如果未提供，请询问。

3. **阅读文档结构——首先是标题。**

   在阅读正文之前，提取：
   - 主协议标题（例如，"Master Services Agreement"、"Non-Disclosure Agreement"）
   - 所有附件、日程表、附录和附件的标题（例如，"Exhibit A — Data Processing Addendum"、"Schedule 1 — Subscription Order Form"、"Annex B — Service Level Agreement"）

   这是路由信号。不要仅依赖正文关键词——带有"confidential"的 40 页 MSA 不是 NDA。

4. **根据文档结构选择 skill。**

   将每个识别的文档或部分映射到 skill：

   | 文档 / 部分标题包含 | Skill |
   |---|---|
   | Non-Disclosure、NDA、Confidentiality Agreement（作为*主*协议） | **nda-review** |
   | Master Services Agreement、Professional Services、Statement of Work、Consulting Agreement | **vendor-agreement-review** |
   | Subscription、SaaS、Cloud Services、带有自动续订的 Order Form、带有经常性费用的 Software License | **saas-msa-review**（vendor-agreement-review 的覆盖层） |
   | Data Processing Addendum、DPA、Data Processing Agreement（作为附件或独立） | **vendor-agreement-review** → 数据保护部分的注释 |
   | Service Level Agreement、SLA（作为附件） | **saas-msa-review** → SLA 部分的注释 |

   可能适用多个 skills。常见组合：
   - MSA + DPA 附件 → vendor-agreement-review，并注明 DPA
   - SaaS 订阅 + Order Form + SLA 附件 → saas-msa-review（覆盖所有三个）
   - MSA + 带有自动续订的 Order Form → vendor-agreement-review + saas-msa-review 覆盖层

   当阅读标题后结构真正模糊时（例如，标题为"Agreement"且未列出附件的文档），阅读正文的前两页以解决它——然后停止并路由。

5. **如果启用，则确认路由。**

   如果 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 中的 `confirm_routing` 为 `true`（或字段不存在）：

   ```
   我打算将其审查为：[agreement type(s)]。

   已识别的文档：
   - [Main agreement title] → [skill]
   - [Exhibit A title] → [将如何处理]
   - [Exhibit B title] → [将如何处理]

   正确吗？（yes / no — 或者告诉我哪里错了）
   ```

   在继续之前等待确认。如果用户更正路由，应用他们的指令并继续。

   如果 `confirm_routing` 为 `false`：静默继续。在审查备忘录顶部记录路由决策，以便用户可以看到应用了什么。

6. **运行 skill。** 完全遵循每个 skill 的工作流。如果适用多个 skills，按顺序运行它们并将输出集成到单个备忘录中——不要生成单独的备忘录。

7. **检查升级：** 如果任何问题根据 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` 矩阵超出审查者的权限，调用 **escalation-flagger** 路由并起草请求。

8. **提供后续选项：**
   - 给业务负责人的利益相关者摘要
   - 带有跟踪修订的红线 .docx
   - [CLM] 记录创建（如果已连接）
   - 添加到续约登记册（如果发现自动续订）

## 配置 confirm_routing

添加到 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` → `## Review preferences`：

```markdown
## Review preferences

confirm_routing: true   # 设置为 false 以跳过路由确认并自动继续
```

冷启动访谈应该询问此偏好。默认为 `true`——已确认开启。随着信任建立，用户可以将其设置为 `false`。

## 示例

```
/commercial-legal:review vendor-msa.pdf
```

```
/commercial-legal:review https://drive.google.com/file/d/ABC123
```

```
/commercial-legal:review
[paste agreement text]
```

## 输出

根据 skill 格式的完整审查备忘录。路由决策记录在顶部。偏差逐偏差、具体红线语言、命名审批人。保存在 `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md` → 内部风格说工作产品去的地方。
