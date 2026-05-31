---
name: customize
description: >
  引导式自定义您的监管执业档案——更改一件事而不重新运行整个冷启动访谈。调整监视的监管机构、
  策略库索引、重要性阈值、差距响应流程、订阅源配置或事项工作区路径。当用户说"更改我的 [某事]"、
  "添加监管机构"、"更新我的监视列表"、"编辑我的阈值"或"自定义"时使用。
argument-hint: "[section name, or describe what you want to change]"
---

<!--
This file is a Chinese translation of the original by Anthropic PBC.
Original: https://github.com/anthropics/claude-for-legal
Licensed under Apache License 2.0
-->


# /customize

## 何时运行

用户输入了 `/regulatory-legal:customize`。他们想更改监管档案中的某些内容——
监视的监管机构、重要性阈值、订阅源来源——而不重新运行整个冷启动访谈且无需手动编辑 YAML。

## 做什么

1. **读取配置。** 读取
   `~/.claude/plugins/config/claude-for-legal/regulatory-legal/CLAUDE.md`
   （以及上一层级的 `~/.claude/plugins/config/claude-for-legal/company-profile.md`）。
   如果插件配置不存在或仍包含 `[PLACEHOLDER]` 值，说：

   > You haven't run setup yet. Run `/regulatory-legal:cold-start-interview`
   > first — customize is for adjusting a profile you already have.

2. **显示可自定义地图。** 列出档案中的内容，分组，附带当前值的一行摘要：

   - **Company / who you are** — 姓名、行业、司法管辖区、阶段、执业
     设置 *(在所有 12 个插件之间共享——更改通过 `company-profile.md` 流动)*
   - **Regulators we watch** —范围内的机构、委员会、SRO、州监管机构，
     以及哪些是"主导"（最有可能驱动策略影响）vs. "监控"
   - **Policy library** —库索引的内部策略、每个的路径、每个策略的所有者
   - **Materiality threshold** —监管变更何时上升到"notable" vs. "report" vs. "digest only"；
     此阈值如何过滤 `/reg-feed-watcher` 输出
   - **Gap response process** —谁分类、每个严重性的 SLA、下游所有者（策略、产品、培训）
   - **Feed configuration** —监管机构订阅源、付费订阅源连接器、
     `/reg-feed-watcher` 扫描的节奏、摘要频道
   - **People** —监管律师、策略所有者、评论起草者、升级链
   - **Workflow** —事项工作区、开放差距跟踪器、评论截止日期跟踪器、摘要发布节奏
   - **Integrations** —监管订阅源 / Slack / 文档存储状态、备用方案

3. **询问他们想要更改什么。**

   > What would you like to adjust? Pick a section, or describe the change in
   > your own words.

4. **进行更改。** 显示当前值，询问新值，解释下游的更改，确认，将其写入配置。

   示例：
   - *向监视列表添加监管机构：* "`/reg-feed-watcher` 将在其下次运行时扫描此
     监管机构。`/policy-diff` 将接受来自此
     监管机构规则制定订阅源的输入。"
   - *收紧重要性阈值：* "`/reg-feed-watcher` 摘要将
     更短——低于新阈值的项目将从每周
     摘要中删除但保持可搜索。"
   - *向库添加新策略：* "`/policy-diff` 将在将新规则与库匹配时包括此策略。评论跟踪器
     将标记影响此策略的评论。"

5. **对于共享档案更改**（company name、industry、jurisdictions、practice setting、stage）：
   写入 `~/.claude/plugins/config/claude-for-legal/company-profile.md` 并注意：

   > This change affects all 12 plugins — any plugin that reads your
   > jurisdiction footprint now sees [new value].

6. **Close.**

   > Done. Your next output will reflect the change. Anything else? You can
   > run `/regulatory-legal:customize` anytime.

## 护栏

- **Never delete a section.** 如果用户想要"删除"监管机构，提议将其标记为 `[Monitor only] 并
  说明监控使订阅源保持在档案中，但将其从活动摘要中拉出。
- **Flag internal inconsistency.** 如果更改会使档案不一致（例如，范围内的监管机构 +
    足迹中监管机构涵盖的司法管辖区；或"weekly digest" + materiality threshold
    导致每季度少于一个项目），标记紧张关系。
- **Flag guardrail degradation.** 引用法规上的 `[verify]` 标记、订阅源拉取上的来源归因以及
    差距分类上的 `[review]` 标记是承重的——不要删除。Materiality threshold 可以调整，
    但将其降低到摘要变成噪音的以下时就是这一点——如果是那个方向则警告。
- **One change at a time.** 不要重新询问整个访谈。
