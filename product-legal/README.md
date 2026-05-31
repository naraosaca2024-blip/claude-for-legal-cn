<!--
This file is a Chinese translation of the original by Anthropic PBC.
Original: https://github.com/anthropics/claude-for-legal
Licensed under Apache License 2.0
-->

# Product Counsel Plugin

产品法律工作流：发布审查、营销声明审查、功能风险评估和快速"这是个问题吗？"分类。围绕从你实际发布审查历史中学到的风险校准构建——在*你的*公司阻止什么，而不是通用的。

**每个输出都是供律师审查的草稿——带引用、标记和把关——而不是法律结论。** plugin 完成工作：阅读文档、应用你的剧本、发现问题、起草备忘录。律师审查、验证并决定。引用按来源标记，以便你知道哪些来自研究工具，哪些需要检查。特权标记保守应用，因此不会意外放弃任何东西。重要行动——提交、发送、执行——在明确确认后进行。

## Who this is for

| 角色 | 主要工作流 |
|---|---|
| **产品法律顾问** | 发布审查、功能风险评估、校准维护 |
| **产品经理** | "这是个问题吗？"分类自助服务 |
| **营销** | 发布前的声明审查 |
| **GC / 法律领导层** | 升级项目的功能风险评估 |

## First run: the cold-start interview

连接到你的发布跟踪器（Jira/Linear），阅读你过去的十个发布审查，了解你实际阻止什么与你放行什么。构建每个其他 skill 都读取的风险校准表。

你的配置存储在 `~/.claude/plugins/config/claude-for-legal/product-legal/CLAUDE.md`，并在 plugin 更新后保留。

```
/product-legal:cold-start-interview
```

## Commands

| 命令 | 功能 |
|---|---|
| `/product-legal:cold-start-interview` | 冷启动访谈 |
| `/product-legal:launch-review [PRD or ticket]` | 根据你的框架进行完整发布审查 |
| `/product-legal:marketing-claims-review [copy]` | 营销声明审查 |
| `/product-legal:is-this-a-problem [question]` | 快速"这是个问题吗？"答案 |
| `/product-legal:matter-workspace` | 管理事项工作区（仅多客户私人执业）——新建、列表、切换、关闭、无 |

## Skills

| Skill | 目的 |
|---|---|
| **cold-start-interview** | 从访谈 + 过去发布审查编写 ~/.claude/plugins/config/claude-for-legal/product-legal/CLAUDE.md |
| **launch-review** | 按类别审查，针对你的公司校准 |
| **marketing-claims-review** | 声明分类：夸张/事实/比较/暗示/绝对 |
| **feature-risk-assessment** | 当发布审查不够时，对一个问题进行深入探讨 |
| **is-this-a-problem** | 快速回答 Slack 问题 |
| **matter-workspace** | 为多客户执业创建、列出、切换和关闭事项工作区；隔离每个客户/事项，因此上下文不会在它们之间泄露 |

## Interactive commands vs. scheduled agents

上面的命令在你调用时运行——用于当你处理事项时。下面的 agent 按计划运行——用于当你不注意时发生变化的内容：

| Agent | 监视内容 | 默认节奏 |
|---|---|---|
| **launch-watcher** | 发布跟踪器（Jira/Linear）中可能需要法律审查的即将发布；根据校准表过滤未来 30 天内有发布日期的工单 | 每日 |

## Integrations

**首先连接研究工具——引用护栏依赖它。** 没有它，每个引用都被标记为 `[verify]`，并且每个可交付成果上方的审查者备注记录来源未经验证。Skill 都可以工作；研究工具（CourtListener）只是将验证工作从你的盘子上移开。

附带在 `.mcp.json` 中配置的连接器：

- **Slack** — 搜索消息、阅读频道、在你的工作区中查找讨论（通用存储桶）
- **Google Drive** — 从 Google Drive 搜索、阅读和获取文档（通用存储桶）
- **Linear** — 问题跟踪和项目管理
- **Atlassian** — Jira 问题和 Confluence 页面
- **Asana** — 任务和项目跟踪

连接跟踪器后：冷启动提取发布历史，发布审查提取工单上下文，发布观察 agent 监视日历。

## Quick start

```
/product-legal:cold-start-interview
```

然后：

```
/product-legal:is-this-a-problem "Can we A/B test the pricing page?"
```

→ 根据你的风险表的即时答案。

```
/product-legal:launch-review PROJ-1234
```

→ 完整审查，按类别，带行动项。

## How it learns

你在 `~/.claude/plugins/config/claude-for-legal/product-legal/CLAUDE.md` 的执业档案不是静态的——它会随着你使用 plugin 而改进。Skill 会告诉你何时输出使用了你应该调整的默认值。你可以重新运行设置、直接编辑文件，或者告诉 skill 记录新立场。

## Notes

- 校准表就是全部。如果错了，每次审查都错。当你的风险姿态改变时（新监管机构、新同意令、新 GC），重新运行设置。
- `is-this-a-problem` 是为 PM 自助服务设计的。它快速回答，并在应该时路由到真实审查。
- 功能风险评估是为 10% 需要深度的发布准备的。大多数不需要——不要生成文书工作。

## Prerequisites

一些功能引用外部集成（文档管理、发布跟踪器、eDiscovery、案件管理、监管源）。这些不捆绑——如果你在环境中有这些中的一个的 MCP 服务器，相关功能将使用它。没有它，plugin 回退到文件上传和手动工作流。运行 `/product-legal:cold-start-interview --check-integrations` 以查看你的环境中可用什么。

## Configuration

你的配置存储在 `~/.claude/plugins/config/claude-for-legal/product-legal/CLAUDE.md`，并在 plugin 更新后保留——你只需运行一次设置。
