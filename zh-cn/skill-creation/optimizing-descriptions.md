---
title: "优化技能描述"
layout: default
---

技能只有在被激活时才有帮助。`SKILL.md` 前置元数据中的 `description` 字段是代理决定是否为特定任务加载技能的主要机制。描述过于笼统意味着技能应该触发时不会触发；描述过于宽泛意味着不应该触发时会触发。

本指南涵盖如何系统化测试和改进技能描述以提高触发准确性。

## 技能触发的工作原理

代理使用[渐进式披露]({{ site.baseurl }}/what-are-skills.html#how-skills-work)来管理上下文。启动时，它们只加载每个可用技能的 `name` 和 `description`——刚好够判断技能何时可能相关。当用户的任务与描述匹配时，代理将完整的 `SKILL.md` 读入上下文并遵循其指令。

这意味着描述承担了触发的全部责任。如果描述没有传达技能何时有用，代理就不会知道要使用它。

一个重要的细微差别：代理通常只为需要超出它们单独处理能力的知识或能力的任务咨询技能。像"读取这个 PDF"这样简单的单步请求可能不会触发 PDF 技能，即使描述完美匹配，因为代理可以用基本工具处理它。涉及专业知识——不熟悉的 API、特定领域的工作流程或不常见格式——的任务，才是精心编写的描述可以发挥作用的地方。

## 编写有效的描述

在测试之前，了解好的描述是什么样的很有帮助。几个原则：

- **使用祈使语气。** 将描述框架为给代理的指令："当...时使用此技能"，而不是"此技能做..."。代理在决定是否行动，所以告诉它何时行动。
- **关注用户意图，而非实现。** 描述用户想要达成什么，而不是技能的内部机制。代理根据用户的请求进行匹配。
- **宁可强势一点。** 明确列出技能适用的上下文，包括用户没有直接命名领域的情况："即使他们没有明确提到 'CSV' 或 '分析'。"
- **保持简洁。** 几句话到一小段通常合适——长到覆盖技能范围，短到不会在代理跨多个技能时过度膨胀上下文。[规范]({{ site.baseurl }}/specification.html#description-field)强制执行 1024 个字符的硬性限制。

## 设计触发评估查询

要测试触发，你需要一组评估查询——标注了应该或不应该触发技能的真实用户提示词。

```json eval_queries.json
[
  { "query": "I've got a spreadsheet in ~/data/q4_results.xlsx with revenue in col C and expenses in col D — can you add a profit margin column and highlight anything under 10%?", "should_trigger": true },
  { "query": "whats the quickest way to convert this json file to yaml", "should_trigger": false }
]
```

目标是大约 20 个查询：8-10 个应该触发，8-10 个不应该触发。

### 应该触发查询

这些测试描述是否捕获了技能的范围。沿多个维度变化：

- **措辞**：一些正式、一些随意、一些有拼写错误或缩写。
- **明确性**：一些直接命名技能领域（"分析这个 CSV"），另一些描述需求而不命名（"我老板要这个数据文件的图表"）。
- **详细程度**：混合简洁提示词和上下文丰富的——一个简短的"分析我的销售 CSV 并制作图表"，旁边是包含文件路径、列名称和背景故事的较长消息。
- **复杂性**：变化步骤数量和决策点。包含单步任务和多步工作流，测试代理是否能识别出技能在更大链路中相关的任务。

最有价值的应该触发查询是那些技能会有帮助但从查询本身看不出关联的查询。这些是描述措辞产生差异的情况——如果查询已经明确要求技能所做之事，任何合理的描述都会触发。

### 不应该触发查询

最有价值的负面测试案例是**近似匹配**——与你的技能共享关键词或概念但实际上需要不同东西的查询。这些测试描述是否精确，而不仅仅是宽泛。

对于 CSV 分析技能，弱的负面示例会是：

- `"Write a fibonacci function"` —— 明显不相关，测试不了什么。
- `"What's the weather today?"` —— 没有关键词重叠，太容易。

强的负面示例：

- `"I need to update the formulas in my Excel budget spreadsheet"` —— 共享"电子表格"和"数据"概念，但需要 Excel 编辑，不是 CSV 分析。
- `"can you write a python script that reads a csv and uploads each row to our postgres database"` —— 涉及 CSV，但任务是数据库 ETL，不是分析。

### 真实感技巧

真实用户提示词包含通用测试查询缺乏的上下文。包含：

- 文件路径（`~/Downloads/report_final_v2.xlsx`）
- 个人背景（`"my manager asked me to..."`）
- 具体细节（列名、公司名、数据值）
- 随意的语言、缩写和偶尔的拼写错误

## 测试描述是否触发

基本方法：在安装了技能的情况下通过代理运行每个查询，观察代理是否调用它。确保技能已注册并可被你的代理发现——具体方式因客户端而异（例如，技能目录、配置文件或 CLI 标志）。

大多数代理客户端提供某种形式的可观察性——执行日志、工具调用历史或详细输出——让你看到运行期间咨询了哪些技能。查看你客户端的文档了解详情。如果代理加载了技能的 `SKILL.md`，技能就触发了；如果代理在没有咨询它的情况下继续，就没有触发。

查询"通过"如果：
- `should_trigger` 为 `true` 且技能被调用，或
- `should_trigger` 为 `false` 且技能未被调用。

### 多次运行

模型行为是不确定的——同一查询可能一次运行触发技能但下次不触发。每个查询运行多次（3 次是合理的起点）并计算**触发率**：技能被调用的运行比例。

应该触发查询通过如果其触发率高于阈值（0.5 是合理的默认值）。不应该触发查询通过如果其触发率低于该阈值。

20 个查询各 3 次运行，那就是 60 次调用。你需要脚本化这个过程。这是一般结构——用你代理客户端提供的替换 `check_triggered` 中的 `claude` 调用和检测逻辑：

```bash
#!/bin/bash
QUERIES_FILE="${1:?Usage: $0 <queries.json>}"
SKILL_NAME="my-skill"
RUNS=3

# 此示例使用 Claude Code 的 JSON 输出来检查 Skill 工具调用。
# 用你代理客户端的检测逻辑替换此函数。
# 如果技能被调用应返回 0（成功），否则返回 1。
check_triggered() {
  local query="$1"
  claude -p "$query" --output-format json 2>/dev/null \
    | jq -e --arg skill "$SKILL_NAME" \
      'any(.messages[].content[]; .type == "tool_use" and .name == "Skill" and .input.skill == $skill)' \
      > /dev/null 2>&1
}

count=$(jq length "$QUERIES_FILE")
for i in $(seq 0 $((count - 1))); do
  query=$(jq -r ".[$i].query" "$QUERIES_FILE")
  should_trigger=$(jq -r ".[$i].should_trigger" "$QUERIES_FILE")
  triggers=0

  for run in $(seq 1 $RUNS); do
    check_triggered "$query" && triggers=$((triggers + 1))
  done

  jq -n \
    --arg query "$query" \
    --argjson should_trigger "$should_trigger" \
    --argjson triggers "$triggers" \
    --argjson runs "$RUNS" \
    '{query: $query, should_trigger: $should_trigger, triggers: $triggers, runs: $runs, trigger_rate: ($triggers / $runs)}'
done | jq -s '.'
```

> **提示：** 如果你的代理客户端支持，可以在结果明确后提前停止运行——代理要么咨询了技能，要么在没有它的情况下开始工作。这可以显著减少运行完整评估集的时间和成本。

## 避免训练/验证集划分的过拟合

如果你针对所有查询优化描述，就有过拟合的风险——精心设计对特定措辞有效的描述但在新措辞上失败。

解决方案是划分查询集：

- **训练集（约 60%）**：用于识别失败和指导改进的查询。
- **验证集（约 40%）**：留出的仅用于检查改进是否泛化的查询。

确保两个集合都包含比例相当的应该触发和不应该触发查询——不要不小心把所有正面例子放在一个集合。随机打乱并保持划分在迭代间固定，这样你在进行同类比较。

如果你使用[上面](#多次运行)那样的脚本，可以将查询分成两个文件——`train_queries.json` 和 `validation_queries.json`——分别对每个运行脚本。

## 优化循环

1. **评估**当前描述在*训练集和验证集*上的表现。训练结果指导你的修改；验证结果告诉你这些修改是否在泛化。
2. **识别**训练集中的失败：哪些应该触发查询没有触发？哪些不应该触发查询触发了？
   - 只用训练集失败来指导修改——无论你是自己修改描述还是提示 LLM，把验证集结果排除在过程之外。
3. **修订描述。** 专注于泛化：
   - 如果应该触发查询失败，描述可能太窄。扩大范围或添加技能何时有用的上下文。
   - 如果不应该触发查询误触发，描述可能太宽。添加关于技能*不*做什么的具体说明，或澄清此技能与相邻能力的边界。
   - 避免从失败查询中添加特定关键词——那是过拟合。相反，找到这些查询代表的一般类别或概念并解决它。
   - 如果几次迭代后卡住了，尝试对描述采用不同的结构方法，而不是增量微调。不同的框架或句子结构可能在精炼无法突破的地方取得突破。
   - 检查描述保持在 1024 字符限制内——描述在优化过程中容易变长。
4. **重复**步骤 1-3，直到所有*训练集*查询通过或你不再看到有意义的改进。
5. **选择最佳迭代**，依据其验证通过率——*验证集*中通过的查询比例。注意最佳描述可能不是你生成的最后一个；早期迭代的验证通过率可能比后来过拟合训练集的更高。

五次迭代通常足够。如果性能没有改善，问题可能在查询（太简单、太难或标签不当）而非描述。

> **提示：** [`skill-creator`](https://github.com/anthropics/skills/tree/main/skills/skill-creator) 技能可以端到端自动化此循环：它划分评估集、并行评估触发率、使用 Claude 提出描述改进建议，并生成你可以边运行边看的实时 HTML 报告。

## 应用结果

选择最佳描述后：

1. 更新 `SKILL.md` 前置元数据中的 `description` 字段。
2. 验证描述在 [1024 字符限制]({{ site.baseurl }}/specification.html#description-field)内。
3. 验证描述按预期触发。手动尝试几个提示词作为快速健全性检查。对于更严格的测试，编写 5-10 个新查询（混合应该触发和不应该触发）并通过评估脚本运行——这些查询从未参与优化过程，它们给你描述是否泛化的诚实检验。

前后对比：

```yaml
# 之前
description: Process CSV files.

# 之后
description: >
  Analyze CSV and tabular data files — compute summary statistics,
  add derived columns, generate charts, and clean messy data. Use this
  skill when the user has a CSV, TSV, or Excel file and wants to
  explore, transform, or visualize the data, even if they don't
  explicitly mention "CSV" or "analysis."
```

改进后的描述对技能做什么更具体（汇总统计、派生列、图表、清理），对何时应用更宽泛（CSV、TSV、Excel；即使没有明确关键词）。

## 下一步

一旦你的技能可靠触发，你会想评估它是否产生好的输出。请参阅[评估技能输出质量]({{ site.baseurl }}/skill-creation/evaluating-skills.html)，了解如何设置测试用例、评估结果和迭代。