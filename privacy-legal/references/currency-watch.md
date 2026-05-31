<!--
This file is a Chinese translation of the original by Anthropic PBC.
Original: https://github.com/anthropics/claude-for-legal
Licensed under Apache License 2.0
-->

# 隐私法时效性监控

**最后验证：2026-05-10。**

> **⚠️ 时效性检查。** 如果上方的最后验证日期超过 90 天，请将此文件视为过时，并在依赖任何条目前进行验证。过时的监控列表比没有监控列表更糟糕——它看起来是最新的，实际上却是错误的。当 skill 读取此文件时，应先检查最后验证日期。如果过时，请说："时效性监控最后验证于 [日期]——[N] 个月前。我将其用作需要搜索的领域清单，而非当前状态的来源。"当你更新任何条目时，也要更新顶部的最后验证日期。

隐私法在不断变动。在依赖生效日期、阈值或义务之前，请进行验证。以下是自模型训练以来最可能已发生变化的领域：

## COPPA（16 CFR Part 312）

- **2025 年修正案——合规截止日期 2026 年 4 月 22 日。** 重大变化：生物特征标识符和政府颁发的身份证件现在是"个人信息"；针对定向广告的第三方披露需要单独的可验证父母同意；强制要求书面信息安全计划；禁止无限期保留。
- 了解 2025 年前 COPPA 的 plugin 看起来很称职，实际上却已过时。请在 [FTC COPPA 页面](https://www.ftc.gov/legal-library/browse/rules/childrens-online-privacy-protection-rule-coppa) 进行验证。

## 州隐私法（综合性）

地图每年都在扩大。截至 2026 年 5 月，已生效或即将生效的综合隐私法所在州：CA（CCPA/CPRA）、VA、CO、CT、UT、IA、IN、TN、MT、OR、TX、FL、DE、NH、NJ、KY、MD、MN、NE、RI。请查看 IAPP 州法律追踪器了解当前生效日期和最新增补。

## 跨境数据传输

- **EU-US DPF** 自 2023 年 7 月起生效。面临 Schrems III 诉讼——在依赖之前请验证其仍然有效。
- **UK-US Data Bridge** 自 2023 年 10 月起生效。
- **Swiss-US DPF** 自 2024 年 9 月起生效。
- 对于任何依赖充分性决定的传输，请查看欧盟委员会当前的充分性决定列表。

## FTC 执法趋势

- **FTC 诉 Humor Rainbow/OkCupid（2026 年 3 月）：** 未披露将用户数据共享给第三方用于 AI 训练属于 § 5 违规。在任何涉及 AI 训练路径的 DPA 或隐私政策审查中需要标记。
- 健康数据：FTC 对健康违规通知规则的扩张性解读（GoodRx、BetterHelp、Premom 和解案）。请验证当前范围。
- 暗模式：FTC 将令人困惑的同意流程视为欺骗性行为的模式。请验证当前执法立场。

## DSAR 响应时限

CCPA：45 天 + 经通知后 45 天延期。GDPR：1 个月 + 2 个月延期。其他州各有不同——请验证特定州的时间窗口。该 plugin 的默认值对于最新的州可能已过时。

## 如何使用此文件

当 skill 引用隐私规则、生效日期或阈值时，应注明："隐私法在不断变动——自我训练以来可能已有变化。请在 [来源] 处验证。参见 `references/currency-watch.md`。"

**此文件会过时。** 截至 2026 年 5 月当前有效。发现偏差时请更新。
