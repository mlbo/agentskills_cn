---
title: "评估技能输出质量"
layout: default
---

你编写了一个技能，用一个提示词测试了一下，看起来能用。但它是否可靠？在不同的提示词下、在边界情况下，它是否比没有技能时表现更好？运行结构化评估（evals）可以回答这些问题，并为你提供系统性改进技能的反馈循环。

## 设计测试用例

一个测试用例包含三个部分：

- **提示词**：一个真实的用户消息——就是用户实际会输入的那种内容。
- **预期输出**：对成功标准的人类可读描述。
- **输入文件**（可选）：技能需要处理的文件。

将测试用例存储在技能目录内的 `evals/evals.json` 中：

```json
{
  "skill_name": "csv-analyzer",
  "evals": [
    {
      "id": 1,
      "prompt": "I have a CSV of monthly sales data in data/sales_2025.csv. Can you find the top 3 months by revenue and make a bar chart?",
      "expected_output": "A bar chart image showing the top 3 months by revenue, with labeled axes and values.",
      "files": ["evals/files/sales_2025.csv"]
    },
    {
      "id": 2,
      "prompt": "there's a csv in my downloads called customers.csv, some rows have missing emails — can you clean it up and tell me how many were missing?",
      "expected_output": "A cleaned CSV with missing emails handled, plus a count of how many were missing.",
      "files": ["evals/files/customers.csv"]
    }
  ]
}
```

**编写好的测试提示词的技巧：**

- **从 2-3 个测试用例开始。** 在看到第一轮结果之前不要过度投入。你可以稍后扩展测试集。
- **变化提示词的风格。** 使用不同的措辞、详细程度和正式程度。有些提示词应该随意一些（"hey can you clean up this csv"），有些则应该精确（"Parse the CSV at data/input.csv, drop rows where column B is null, and write the result to data/output.csv"）。
- **覆盖边界情况。** 至少包含一个测试边界条件的提示词——格式错误的输入、不寻常的请求，或者技能指令可能存在歧义的情况。
- **使用真实的上下文。** 真实用户会提到文件路径、列名和个人背景。"process this data" 这样的提示词太模糊，无法测试任何有价值的内容。

暂时不用担心定义具体的通过/失败检查——只需要提示词和预期输出。在看到第一轮运行产生什么结果后，你再添加详细的检查（称为断言）。

## 运行评估

核心模式是将每个测试用例运行两次：一次**使用技能**，一次**不使用技能**（或使用之前的版本）。这为你提供了一个可对比的基线。

### 工作区结构

在技能目录旁边的工作区目录中组织评估结果。每次完整的评估循环都会获得自己的 `iteration-N/` 目录。在其中，每个测试用例都有一个评估目录，包含 `with_skill/` 和 `without_skill/` 子目录：

```
csv-analyzer/
├── SKILL.md
└── evals/
    └── evals.json
csv-analyzer-workspace/
└── iteration-1/
    ├── eval-top-months-chart/
    │   ├── with_skill/
    │   │   ├── outputs/       # 运行产生的文件
    │   │   ├── timing.json    # token 和耗时
    │   │   └── grading.json   # 断言结果
    │   └── without_skill/
    │       ├── outputs/
    │       ├── timing.json
    │       └── grading.json
    ├── eval-clean-missing-emails/
    │   ├── with_skill/
    │   │   ├── outputs/
    │   │   ├── timing.json
    │   │   └── grading.json
    │   └── without_skill/
    │       ├── outputs/
    │       ├── timing.json
    │       └── grading.json
    └── benchmark.json         # 汇总统计数据
```

你需要手工编写的主要文件是 `evals/evals.json`。其他 JSON 文件（`grading.json`、`timing.json`、`benchmark.json`）是在评估过程中生成的——由代理、脚本或你本人生成。

### 启动运行

每次评估运行都应从干净的上下文开始——没有之前运行或技能开发过程留下的状态。这确保代理只遵循 `SKILL.md` 中的指令。在支持子代理的环境中（例如 Claude Code），这种隔离是自然的：每个子任务都从全新状态开始。如果没有子代理，请为每次运行使用单独的会话。

对于每次运行，提供：

- 技能路径（或基线运行时不提供技能）
- 测试提示词
- 任何输入文件
- 输出目录

以下是你为单次使用技能运行给代理的指令示例：

```
Execute this task:
- Skill path: /path/to/csv-analyzer
- Task: I have a CSV of monthly sales data in data/sales_2025.csv.
  Can you find the top 3 months by revenue and make a bar chart?
- Input files: evals/files/sales_2025.csv
- Save outputs to: csv-analyzer-workspace/iteration-1/eval-top-months-chart/with_skill/outputs/
```

对于基线，使用相同的提示词但不提供技能路径，保存到 `without_skill/outputs/`。

在改进现有技能时，将之前的版本作为基线。在编辑前对其做快照（`cp -r <skill-path> <workspace>/skill-snapshot/`），将基线运行指向该快照，并保存到 `old_skill/outputs/` 而不是 `without_skill/`。

### 捕获计时数据

计时数据让你可以比较技能相对于基线消耗了多少时间和 token——一个大幅提升输出质量但使 token 使用量增加三倍的技能，与既更好又更便宜的技能，是完全不同的权衡。当每次运行完成时，记录 token 数量和耗时：

```json
{
  "total_tokens": 84852,
  "duration_ms": 23332
}
```

> **提示：** 在 Claude Code 中，当子代理任务完成时，[任务完成通知](https://platform.claude.com/docs/en/agent-sdk/typescript#sdk-task-notification-message)包含 `total_tokens` 和 `duration_ms`。立即保存这些值——它们不会持久化到其他任何地方。

## 编写断言

断言是关于输出应包含或达成的内容的可验证声明。在看到第一轮输出后再添加它们——在技能运行之前，你通常不知道什么是"好的"。

好的断言：

- `"The output file is valid JSON"` —— 可通过程序验证。
- `"The bar chart has labeled axes"` —— 具体且可观察。
- `"The report includes at least 3 recommendations"` —— 可计数。

弱的断言：

- `"The output is good"` —— 太模糊，无法评判。
- `"The output uses exactly the phrase 'Total Revenue: $X'"` —— 太脆弱；措辞不同但正确的输出会失败。

并非所有内容都需要断言。有些质量——写作风格、视觉设计、输出是否"感觉对"——很难分解为通过/失败检查。这些最好在[人工审查](#人工审查结果)时发现。将断言保留给可以客观检查的内容。

将断言添加到 `evals/evals.json` 的每个测试用例中：

```json
{
  "skill_name": "csv-analyzer",
  "evals": [
    {
      "id": 1,
      "prompt": "I have a CSV of monthly sales data in data/sales_2025.csv. Can you find the top 3 months by revenue and make a bar chart?",
      "expected_output": "A bar chart image showing the top 3 months by revenue, with labeled axes and values.",
      "files": ["evals/files/sales_2025.csv"],
      "assertions": [
        "The output includes a bar chart image file",
        "The chart shows exactly 3 months",
        "Both axes are labeled",
        "The chart title or caption mentions revenue"
      ]
    }
  ]
}
```

## 评判输出

评判意味着根据实际输出评估每个断言，并记录 **PASS** 或 **FAIL** 及具体证据。证据应引用或参考输出，而不仅仅是陈述观点。

最简单的方法是将输出和断言交给一个 LLM，让它逐一评估。对于可以通过代码检查的断言（有效的 JSON、正确的行数、文件存在且具有预期尺寸），使用验证脚本——脚本对于机械检查比 LLM 判断更可靠，并且可跨迭代复用。

```json
{
  "assertion_results": [
    {
      "text": "The output includes a bar chart image file",
      "passed": true,
      "evidence": "Found chart.png (45KB) in outputs directory"
    },
    {
      "text": "The chart shows exactly 3 months",
      "passed": true,
      "evidence": "Chart displays bars for March, July, and November"
    },
    {
      "text": "Both axes are labeled",
      "passed": false,
      "evidence": "Y-axis is labeled 'Revenue ($)' but X-axis has no label"
    },
    {
      "text": "The chart title or caption mentions revenue",
      "passed": true,
      "evidence": "Chart title reads 'Top 3 Months by Revenue'"
    }
  ],
  "summary": {
    "passed": 3,
    "failed": 1,
    "total": 4,
    "pass_rate": 0.75
  }
}
```

### 评判原则

- **PASS 需要具体证据。** 不要给予假设性的好处。如果断言说"includes a summary"而输出有一个标题为"Summary"的部分但只有一句模糊的话，那就是 FAIL——标签在那里但实质内容没有。
- **审查断言本身，而不仅仅是结果。** 在评判时，注意哪些断言太简单（无论技能质量如何总是通过）、太难（即使输出很好也总是失败）或不可验证（无法仅从输出检查）。在下次迭代中修复这些。

> **提示：** 对于比较两个技能版本，尝试**盲比较**：向 LLM 评判者展示两个输出而不透露哪个来自哪个版本。评判者根据自己的评分标准对整体质量——组织、格式、可用性、精细度——打分，不受哪个版本"应该"更好的偏见影响。这补充了断言评判：两个输出可能都通过所有断言但在整体质量上有显著差异。

## 汇总结果

当迭代中的每次运行都评判完毕后，计算每种配置的汇总统计并保存到评估目录旁边的 `benchmark.json`（例如 `csv-analyzer-workspace/iteration-1/benchmark.json`）：

```json
{
  "run_summary": {
    "with_skill": {
      "pass_rate": { "mean": 0.83, "stddev": 0.06 },
      "time_seconds": { "mean": 45.0, "stddev": 12.0 },
      "tokens": { "mean": 3800, "stddev": 400 }
    },
    "without_skill": {
      "pass_rate": { "mean": 0.33, "stddev": 0.10 },
      "time_seconds": { "mean": 32.0, "stddev": 8.0 },
      "tokens": { "mean": 2100, "stddev": 300 }
    },
    "delta": {
      "pass_rate": 0.50,
      "time_seconds": 13.0,
      "tokens": 1700
    }
  }
}
```

`delta` 告诉你技能的成本（更多时间、更多 token）和收益（更高的通过率）。一个增加 13 秒但将通过率提高 50 个百分点的技能可能是值得的。一个使 token 使用量翻倍但只提高 2 个百分点的技能可能不值得。

> **注意：** 标准差（`stddev`）只有在每次评估有多次运行时才有意义。在只有 2-3 个测试用例且单次运行的早期迭代中，专注于原始通过计数和 delta——随着你扩展测试集并对每个评估多次运行，统计指标才会变得有用。

## 分析模式

汇总统计可能会隐藏重要的模式。在计算基准测试后：

- **删除或替换在两种配置中总是通过的断言。** 这些不会告诉你任何有用的信息——模型在没有技能的情况下也能处理好它们。它们会虚高使用技能的通过率，但不能反映技能的实际价值。
- **调查在两种配置中总是失败的断言。** 要么断言有问题（要求模型做不到的事情），要么测试用例太难，要么断言检查的方向错了。在下次迭代前修复这些。
- **研究使用技能时通过但不使用时失败的断言。** 这是技能明显增加价值的地方。理解*为什么*——哪些指令或脚本产生了差异？
- **当结果在运行间不一致时，收紧指令。** 如果同一个评估有时通过有时失败（在基准测试中表现为高 `stddev`），该评估可能不稳定（对模型随机性敏感），或者技能的指令可能足够模糊以至于模型每次解释不同。添加示例或更具体的指导来减少歧义。
- **检查时间和 token 异常值。** 如果某个评估耗时是其他的三倍，阅读其执行记录（模型在运行期间所做事情的完整日志）来找到瓶颈。

## 人工审查结果

断言评判和模式分析能发现很多问题，但它们只检查你想到写断言的内容。人工审查者带来全新的视角——发现你没有预料到的问题，注意到输出技术上正确但偏离重点，或者发现难以表达为通过/失败检查的问题。对于每个测试用例，审查实际输出以及评判结果。

记录每个测试用例的具体反馈并保存在工作区中（例如，作为评估目录旁边的 `feedback.json`）：

```json
{
  "eval-top-months-chart": "The chart is missing axis labels and the months are in alphabetical order instead of chronological.",
  "eval-clean-missing-emails": ""
}
```

"The chart is missing axis labels" 是可操作的；"looks bad" 不是。空反馈意味着输出看起来没问题——该测试用例通过了你的审查。在[迭代技能](#迭代技能)步骤中，将改进集中在你有具体抱怨的测试用例上。

## 迭代技能

在评判和审查后，你有三个信号来源：

- **失败的断言**指向具体的差距——一个缺失的步骤、一个不清晰的指令，或者一个技能没有处理的情况。
- **人工反馈**指向更广泛的质量问题——方法错误、输出结构不良，或者技能产生了技术上正确但没有帮助的结果。
- **执行记录**揭示事情*为什么*出错。如果代理忽略了一条指令，该指令可能有歧义。如果代理在不高效的步骤上花费时间，这些指令可能需要简化或删除。

将这些信号转化为技能改进的最有效方法是将这三者——连同当前的 `SKILL.md`——交给一个 LLM 并让它提出修改建议。LLM 可以综合失败断言、审查者抱怨和记录行为的模式，这些手动连接会很繁琐。在提示 LLM 时，包含这些指导原则：

- **从反馈中概括。** 技能将用于许多不同的提示词，不仅仅是测试用例。修复应该广泛地解决根本问题，而不是为特定示例添加狭隘的补丁。
- **保持技能精简。** 更少、更好的指令往往优于穷尽的规则。如果记录显示浪费的工作（不必要的验证、不需要的中间输出），删除那些指令。如果通过率在添加更多规则后趋于平稳，技能可能过度约束——尝试删除指令，看看结果是否保持或改善。
- **解释原因。** 基于推理的指令（"Do X because Y tends to cause Z"）比僵化的指令（"ALWAYS do X, NEVER do Y"）效果更好。当模型理解目的时，它能更可靠地遵循指令。
- **打包重复工作。** 如果每次测试运行都独立编写了类似的辅助脚本（图表构建器、数据解析器），那是将脚本打包到技能的 `scripts/` 目录的信号。参见[使用脚本]({{ site.baseurl }}/skill-creation/using-scripts.html)了解如何做到这一点。

### 循环

1. 将评估信号和当前 `SKILL.md` 交给 LLM 并让它提出改进建议。
2. 审查并应用更改。
3. 在新的 `iteration-<N+1>/` 目录中重新运行所有测试用例。
4. 评判并汇总新结果。
5. 人工审查。重复。

当你对结果满意、反馈持续为空、或迭代之间不再看到有意义的改进时停止。

> **提示：** [`skill-creator`](https://github.com/anthropics/skills/tree/main/skills/skill-creator) 技能可以自动化此工作流程的大部分——运行评估、评判断言、汇总基准测试并呈现结果供人工审查。