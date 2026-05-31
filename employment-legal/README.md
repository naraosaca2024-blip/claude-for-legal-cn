<!--
This file is a Chinese translation of the original by Anthropic PBC.
Original: https://github.com/anthropics/claude-for-legal
Licensed under Apache License 2.0
-->

# Employment Counsel Plugin

内部劳动法律工作流：招聘审查、解雇审查、政策起草、员工手册更新、司法管辖区感知的工资与工时问答。围绕在冷启动时学习的司法管辖区足迹构建——plugin 知道你所在的州以及每个州的不同之处。

**每个输出都是供律师审查的草稿——带引用、标记和把关——而不是法律结论。** plugin 完成工作：阅读文档、应用你的剧本、发现问题、起草备忘录。律师审查、验证并决定。引用按来源标记，以便你知道哪些来自研究工具，哪些需要检查。特权标记保守应用，因此不会意外放弃任何东西。重要行动——提交、发送、执行——在明确确认后进行。

## Who this is for

| 角色 | 主要工作流 |
|---|---|
| **劳动法律顾问** | 解雇审查、政策起草、工资/工时分析 |
| **HR 业务合作伙伴** | 招聘审查、员工手册问题、一线工资/工时问答 |
| **GC** | 高风险条款和裁员的升级接收方 |

## First run: cold-start

询问你在哪些州和国家有员工，阅读你的员工手册和三份最近的解雇备忘录，构建司法管辖区感知的升级表格。

```
/employment-legal:cold-start-interview
```

你的配置存储在 `~/.claude/plugins/config/claude-for-legal/employment-legal/CLAUDE.md`，并在 plugin 更新后保留。

## Prerequisites

- **持久数据路径。** 请假登记、调查日志和扩展跟踪器写入 `~/.claude/plugins/config/claude-for-legal/employment-legal/`，这是一个版本无关的路径，在 plugin 更新后保留。这些文件包含特权和敏感的人员信息——确保该目录已备份并受到访问控制。
- **法律研究访问。** 此 plugin 中的 skill 有意不存储实体法律规则（工资阈值、限制性契约可执行性、最终工资时间、解除协议考虑期、特定国家的雇佣框架等）。每个司法管辖区特定的规则在审查时进行研究和引用。确保会话可以访问你依赖的研究工具（网页搜索、内部法律研究集成、团队参考材料）。
- **外部法律顾问。** 在没有外部法律顾问参与任何紧急情况或新司法管辖区的情况下，不会产生任何国家特定或司法管辖区特定的法律建议。

## Skills

| Skill | 功能 |
|---|---|
| `/employment-legal:cold-start-interview` | 冷启动访谈——从员工手册+解雇备忘录中学习司法管辖区足迹+升级规则 |
| `/employment-legal:hiring-review` | 录用函+限制性契约审查，司法管辖区检查 |
| `/employment-legal:termination-review` | 带有高风险标志检测的解雇审查 |
| `/employment-legal:policy-drafting [topic]` | 在需要时起草带有州补充的政策 |
| `/employment-legal:wage-hour-qa [question]` | 工资/工时或一般雇佣问答，司法管辖区感知 |
| `/employment-legal:worker-classification` | 对拟议的工人参与进行分类，并标记错误分类差距 |
| `/employment-legal:expansion-kickoff [country]` | 启动新国家的国际扩展规划 |
| `/employment-legal:expansion-update [country]` | 更新进行中的扩展跟踪器 |
| `/employment-legal:investigation-open` | 打开新的内部调查事项 |
| `/employment-legal:investigation-add` | 向正在进行的调查添加文档、访谈笔记或观察结果 |
| `/employment-legal:investigation-query` | 针对正在进行的调查日志提问 |
| `/employment-legal:investigation-memo` | 起草或更新特权调查备忘录 |
| `/employment-legal:investigation-summary` | 从调查备忘录中起草针对特定受众的摘要 |
| `/employment-legal:leave-tracker` | 检查未结请假的截止日期提醒和所需决定 |
| `/employment-legal:log-leave` | 将新请假添加到请假登记 |
| `/employment-legal:matter-workspace` | 管理事项工作区（仅多客户私人执业）——新建、列表、切换、关闭、无 |
| **handbook-updates** | 将拟议变更与当前员工手册进行比较，标记州补充影响 |

参考 skill `internal-investigation` 和 `international-expansion` 包含详细框架和模板——上面的按模式 skill 根据需要加载它们。

## Interactive skills vs. scheduled agents

上面的 skill 在你调用时运行——用于当你处理事项时。下面的 agent 按计划运行——用于当你不注意时发生变化的内容：

| Agent | 监视内容 | 默认节奏 |
|---|---|---|
| **leave-tracker** | 具有严格法律截止日期的未结请假——FMLA、州等效（CA CFRA、NY PFL）、USERRA、作为调整的 ADA 请假；在错过截止日期前触发决策点提醒 | 每周（周一） |

## How it learns

你在 `~/.claude/plugins/config/claude-for-legal/employment-legal/CLAUDE.md` 的执业档案不是静态的——它会随着你使用 plugin 而改进。Skill 会告诉你何时输出使用了你应该调整的默认值。你可以重新运行设置、直接编辑文件，或者告诉 skill 记录新立场。

## Notes

- 司法管辖区感知是关键。Plugin 知道加利福尼亚的最终工资应在最后一天到期，纽约的是下一个发薪日。
- 解雇审查不是与 HR 和经理对话的替代品。它是一个检查清单，可以捕捉每个人都忘记的事情。
- 工资/工时问答引用规则但标记紧急情况供人工审查。分类决定有后果。
