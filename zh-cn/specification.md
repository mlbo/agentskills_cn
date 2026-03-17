---
title: "规范"
layout: default
---

## 目录结构

技能是一个目录，至少包含一个 `SKILL.md` 文件：

```
skill-name/
├── SKILL.md          # 必需：元数据 + 指令
├── scripts/          # 可选：可执行代码
├── references/       # 可选：文档
├── assets/           # 可选：模板、资源
└── ...               # 任何其他文件或目录
```

## `SKILL.md` 格式

`SKILL.md` 文件必须包含 YAML 前置元数据，后跟 Markdown 内容。

### 前置元数据

| 字段 | 必需 | 约束 |
|-------|----------|-------------|
| `name` | 是 | 最多 64 个字符。仅限小写字母、数字和连字符。不能以连字符开头或结尾。 |
| `description` | 是 | 最多 1024 个字符。非空。描述技能的功能及使用场景。 |
| `license` | 否 | 许可证名称或对捆绑许可证文件的引用。 |
| `compatibility` | 否 | 最多 500 个字符。指示环境要求（目标产品、系统包、网络访问等）。 |
| `metadata` | 否 | 用于附加元数据的任意键值映射。 |
| `allowed-tools` | 否 | 以空格分隔的预批准工具列表，技能可以使用这些工具。（实验性） |

> **最小示例：**
>
> ```markdown SKILL.md
> ---
> name: skill-name
> description: A description of what this skill does and when to use it.
> ---
> ```
>
> **包含可选字段的示例：**
>
> ```markdown SKILL.md
> ---
> name: pdf-processing
> description: Extract PDF text, fill forms, merge files. Use when handling PDFs.
> license: Apache-2.0
> metadata:
>   author: example-org
>   version: "1.0"
> ---
> ```

#### `name` 字段

必需的 `name` 字段：
- 必须为 1-64 个字符
- 只能包含 Unicode 小写字母数字字符（`a-z`）和连字符（`-`）
- 不能以连字符（`-`）开头或结尾
- 不能包含连续的连字符（`--`）
- 必须与父目录名称匹配

> **有效示例：**
> ```yaml
> name: pdf-processing
> ```
> ```yaml
> name: data-analysis
> ```
> ```yaml
> name: code-review
> ```
>
> **无效示例：**
> ```yaml
> name: PDF-Processing  # 不允许大写
> ```
> ```yaml
> name: -pdf  # 不能以连字符开头
> ```
> ```yaml
> name: pdf--processing  # 不允许连续连字符
> ```

#### `description` 字段

必需的 `description` 字段：
- 必须为 1-1024 个字符
- 应同时描述技能的功能和使用场景
- 应包含有助于代理识别相关任务的具体关键词

> **良好示例：**
> ```yaml
> description: Extracts text and tables from PDF files, fills PDF forms, and merges multiple PDFs. Use when working with PDF documents or when the user mentions PDFs, forms, or document extraction.
> ```
>
> **较差示例：**
> ```yaml
> description: Helps with PDFs.
> ```

#### `license` 字段

可选的 `license` 字段：
- 指定应用于技能的许可证
- 我们建议保持简短（许可证名称或捆绑许可证文件的名称）

> **示例：**
> ```yaml
> license: Proprietary. LICENSE.txt has complete terms
> ```

#### `compatibility` 字段

可选的 `compatibility` 字段：
- 如果提供，必须为 1-500 个字符
- 仅当技能有特定环境要求时才应包含
- 可以指示目标产品、所需的系统包、网络访问需求等

> **示例：**
> ```yaml
> compatibility: Designed for Claude Code (or similar products)
> ```
> ```yaml
> compatibility: Requires git, docker, jq, and access to the internet
> ```

> **注意：** 大多数技能不需要 `compatibility` 字段。

#### `metadata` 字段

可选的 `metadata` 字段：
- 从字符串键到字符串值的映射
- 客户端可以使用此字段存储 Agent Skills 规范未定义的附加属性
- 我们建议使键名具有合理的唯一性，以避免意外冲突

> **示例：**
> ```yaml
> metadata:
>   author: example-org
>   version: "1.0"
> ```

#### `allowed-tools` 字段

可选的 `allowed-tools` 字段：
- 以空格分隔的预批准运行工具列表
- 实验性功能。不同代理实现对此字段的支持可能有所不同

> **示例：**
> ```yaml
> allowed-tools: Bash(git:*) Bash(jq:*) Read
> ```

### 正文内容

前置元数据之后的 Markdown 正文包含技能指令。没有格式限制。编写任何有助于代理有效执行任务的内容。

推荐的部分：
- 分步指令
- 输入和输出示例
- 常见边缘情况

请注意，代理一旦决定激活技能，就会加载整个文件。考虑将较长的 `SKILL.md` 内容拆分到引用文件中。

## 可选目录

### `scripts/`

包含代理可以运行的可执行代码。脚本应该：
- 自包含或明确记录依赖项
- 包含有用的错误消息
- 优雅地处理边缘情况

支持的语言取决于代理实现。常见选项包括 Python、Bash 和 JavaScript。

### `references/`

包含代理在需要时可以阅读的附加文档：
- `REFERENCE.md` - 详细技术参考
- `FORMS.md` - 表单模板或结构化数据格式
- 特定领域的文件（`finance.md`、`legal.md` 等）

保持单个[引用文件](#文件引用)的专注性。代理按需加载这些文件，因此较小的文件意味着更少的上下文使用。

### `assets/`

包含静态资源：
- 模板（文档模板、配置模板）
- 图像（图表、示例）
- 数据文件（查找表、模式）

## 渐进式披露

技能的结构应便于高效使用上下文：

1. **元数据**（约 100 个 token）：所有技能在启动时加载 `name` 和 `description` 字段
2. **指令**（建议 < 5000 个 token）：技能激活时加载完整的 `SKILL.md` 正文
3. **资源**（按需）：文件（例如 `scripts/`、`references/` 或 `assets/` 中的文件）仅在需要时加载

将主 `SKILL.md` 保持在 500 行以内。将详细的参考材料移至单独的文件中。

## 文件引用

在技能中引用其他文件时，使用相对于技能根目录的路径：

```markdown SKILL.md
详见[参考指南](references/REFERENCE.md)。

运行提取脚本：
scripts/extract.py
```

保持文件引用从 `SKILL.md` 向下一级。避免深度嵌套的引用链。

## 验证

使用 [skills-ref](https://github.com/agentskills/agentskills/tree/main/skills-ref) 参考库来验证您的技能：

```bash
skills-ref validate ./my-skill
```

这将检查您的 `SKILL.md` 前置元数据是否有效并遵循所有命名约定。