---
title: "什么是技能？"
layout: default
---

从本质上讲，技能是一个包含 `SKILL.md` 文件的文件夹。该文件包含元数据（至少包括 `name` 和 `description`）以及告诉代理如何执行特定任务的指令。技能还可以打包脚本、模板和参考资料。

```
my-skill/
├── SKILL.md          # 必需：指令 + 元数据
├── scripts/          # 可选：可执行代码
├── references/       # 可选：文档
└── assets/           # 可选：模板、资源
```

## 技能如何工作

技能使用**渐进式披露**来高效管理上下文：

1. **发现**：启动时，代理仅加载每个可用技能的名称和描述，刚好足以了解何时可能需要使用它。

2. **激活**：当任务与技能的描述匹配时，代理将完整的 `SKILL.md` 指令读取到上下文中。

3. **执行**：代理遵循指令，根据需要选择性地加载引用的文件或执行打包的代码。

这种方法使代理保持快速，同时允许它们按需访问更多上下文。

## SKILL.md 文件

每个技能都以一个包含 YAML 前置元数据和 Markdown 指令的 `SKILL.md` 文件开始：

```mdx
---
name: pdf-processing
description: 提取 PDF 文本、填写表单、合并文件。在处理 PDF 时使用。
---

# PDF 处理

## 何时使用此技能
当用户需要处理 PDF 文件时使用此技能...

## 如何提取文本
1. 使用 pdfplumber 进行文本提取...

## 如何填写表单
...
```

`SKILL.md` 顶部需要以下前置元数据：

- `name`：简短标识符
- `description`：何时使用此技能

Markdown 正文包含实际指令，对结构或内容没有特定限制。

这种简单的格式具有一些关键优势：

- **自文档化**：技能创建者或用户可以阅读 `SKILL.md` 并理解它的功能，使技能易于审计和改进。

- **可扩展**：技能的复杂程度可以从纯文本指令到可执行代码、资源和模板。

- **可移植**：技能只是文件，因此易于编辑、版本控制和共享。

## 下一步

- [查看规范]({{ site.baseurl }}/specification.html) 以了解完整格式。
- [为您的代理添加技能支持]({{ site.baseurl }}/client-implementation/adding-skills-support.html) 以构建兼容的客户端。
- [在 GitHub 上查看示例技能](https://github.com/anthropics/skills)。
- [阅读编写最佳实践](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) 以编写有效的技能。
- [使用参考库](https://github.com/agentskills/agentskills/tree/main/skills-ref) 验证技能并生成提示词 XML。