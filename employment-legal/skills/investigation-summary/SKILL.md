---
name: investigation-summary
description: >
  从特权调查备忘录起草面向特定受众的摘要——HR、领导层或外部律师版本。当调查备忘录需要传达给不应看到完整特权工作产品的受众时使用。
argument-hint: "[事项名称] [受众: hr / leadership / outside-counsel]"
---

<!--
This file is a Chinese translation of the original by Anthropic PBC.
Original: https://github.com/anthropics/claude-for-legal
Licensed under Apache License 2.0
-->


# /investigation-summary

从特权调查备忘录起草精简的、面向受众的摘要。HR 摘要不包含特权分析。领导层摘要为高级别概览。外部律师简报包含完整上下文。

## 说明

1. 加载 `internal-investigation` 参考 skill 并运行模式 5（受众摘要）。
2. 如果备忘录尚不存在，提供先起草备忘录的选项。
3. HR 摘要不包含律师心理印象、可信度方法或法律敞口分析。

## 示例

```
/employment-legal:investigation-summary [事项名称] hr
```

```
/employment-legal:investigation-summary [事项名称] leadership
```

```
/employment-legal:investigation-summary [事项名称] outside-counsel
```

> 详细的受众剥离规则和摘要模板位于 `internal-investigation` 参考 skill 中——在进行实质性工作之前加载它。
