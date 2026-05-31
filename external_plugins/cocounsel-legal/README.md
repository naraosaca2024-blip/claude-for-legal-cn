<!--
This file is a Chinese translation of the original by Anthropic PBC.
Original: https://github.com/anthropics/claude-for-legal
Licensed under Apache License 2.0
-->

# CoCounsel Legal

CoCounsel Legal 为 CoCounsel Legal 订阅者将 Westlaw 深度研究带入 Claude。跨美国联邦和州法律运行司法管辖区特定的法律研究，并收到带有 Westlaw 和 Practical Law 来源链接的完整引用报告。深度研究报告引用判例法、法规、条例、行政材料、Practical Law、次要来源和当前意识内容。在同一对话中提出后续问题。连接器从 Westlaw 深度研究开始，将随着时间扩展到其他 CoCounsel Legal 功能。

- 使用 CoCounsel Legal 运行 Westlaw 深度研究，获得完整引用报告，包括指向 Westlaw 和 Practical Law 来源的链接引用。
- 在一次研究运行中询问最多三个美国司法管辖区的法律研究。
- 返回已完成的 CoCounsel Legal 研究对话并稍后检索报告。
- 在同一对话中提出后续问题，无需重新开始研究。

## 示例用例

1. 研究自 2020 年以来加州法院如何对待高管的非竞争协议。
2. 我之前让你研究过加州的非竞争协议。你现在能检索完整报告吗？
3. 跟进该研究：同一时期德克萨斯州法律在高管非竞争协议方面有何不同？

## 何时使用

任何可从判例法、法规、条例、行政材料、次要来源、Practical Law 文档和当前意识（包括 JD Supra）回答的问题。示例包括：

- 法院如何对某个问题作出裁决，或什么权威支持或挑战某个立场
- 索赔的元素或抗辩，或特定司法管辖区中某个问题的管辖标准
- 如何解释和应用法规、条例或原则
- 未解决问题双方的论点

## 何时不使用

- 检索特定文档的全文
- 总结特定法规、条例或专著本身所说的内容（例如，"加州证据法典关于传闻证据说了什么？"或"Wright & Miller 关于第 11 条说了什么？"）
- 分析请求（"Scalia 大法官有多少次裁定支持……？"）
- 计算（"如果……，最后可能的提交日期是什么？"）
- 结果预测（"原告在简易判决中胜诉的可能性有多大？"）
- 识别客户可以提出的诉因（skill 研究法律怎么说，而非给定的一组事实是否陈述了索赔）
- 将法律应用于特定事实模式或场景（skill 研究抽象的法律问题，而非法律将如何解决你的事实）
- 起草法律文档、表单或模板
- 关于特定法官、律师或当事人的信息
- 外国或非美国法律
- 执行任务的命令（"给我发送关于 X 案件的电子邮件"）
- 不需要当前权威的一般法律定义
- 跨三个以上司法管辖区的比较
- 布尔搜索查询（工具期望自然语言）


### 链接

- **文档：** https://legal-mcp.thomsonreuters.com/docs/connector-guide
- **支持：** cocounselsupport@tr.com
