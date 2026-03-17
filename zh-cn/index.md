---
title: "概述"
layout: default
---

Agent Skills 是包含指令、脚本和资源的文件夹，代理可以发现并使用它们来更准确、更高效地完成任务。

## 为什么选择 Agent Skills？

代理的能力越来越强，但往往缺乏可靠地完成实际工作所需的上下文。技能通过向代理提供过程性知识以及公司、团队和用户特定的上下文（可按需加载）来解决这个问题。拥有一套技能的代理可以根据正在处理的任务扩展其能力。

**对于技能创建者**：一次构建能力，即可在多个代理产品中部署。

**对于兼容代理**：支持技能可以让最终用户开箱即用地赋予代理新能力。

**对于团队和企业**：以可移植、版本控制的包的形式捕获组织知识。

## Agent Skills 可以实现什么？

- **领域专业知识**：将专业知识打包成可复用的指令，从法律审查流程到数据分析管道。
- **新能力**：赋予代理新能力（例如创建演示文稿、构建 MCP 服务器、分析数据集）。
- **可重复的工作流程**：将多步骤任务转化为一致且可审计的工作流程。
- **互操作性**：在不同的兼容技能的代理产品中复用同一个技能。

## 采用情况

Agent Skills 得到了领先 AI 开发工具的支持。

<div class="logo-grid" style="display: grid; grid-template-columns: repeat(auto-fill, minmax(120px, 1fr)); gap: 24px; align-items: center; justify-items: center; margin: 24px 0;">
  <a href="https://junie.jetbrains.com/" title="Junie"><img src="{{ site.baseurl }}/images/logos/junie/junie-logo-on-white.svg" alt="Junie" style="max-width: 100px; max-height: 40px; object-fit: contain;"></a>
  <a href="https://geminicli.com" title="Gemini CLI"><img src="{{ site.baseurl }}/images/logos/gemini-cli/gemini-cli-logo_light.svg" alt="Gemini CLI" style="max-width: 100px; max-height: 40px; object-fit: contain;"></a>
  <a href="https://cursor.com/" title="Cursor"><img src="{{ site.baseurl }}/images/logos/cursor/LOCKUP_HORIZONTAL_2D_LIGHT.svg" alt="Cursor" style="max-width: 100px; max-height: 40px; object-fit: contain;"></a>
  <a href="https://claude.ai/code" title="Claude Code"><img src="{{ site.baseurl }}/images/logos/claude-code/Claude-Code-logo-Slate.svg" alt="Claude Code" style="max-width: 100px; max-height: 40px; object-fit: contain;"></a>
  <a href="https://claude.ai/" title="Claude"><img src="{{ site.baseurl }}/images/logos/claude-ai/Claude-logo-Slate.svg" alt="Claude" style="max-width: 100px; max-height: 40px; object-fit: contain;"></a>
  <a href="https://github.com/" title="GitHub"><img src="{{ site.baseurl }}/images/logos/github/GitHub_Lockup_Dark.svg" alt="GitHub" style="max-width: 100px; max-height: 40px; object-fit: contain;"></a>
  <a href="https://code.visualstudio.com/" title="VS Code"><img src="{{ site.baseurl }}/images/logos/vscode/vscode.svg" alt="VS Code" style="max-width: 100px; max-height: 40px; object-fit: contain;"></a>
  <a href="https://developers.openai.com/codex" title="OpenAI Codex"><img src="{{ site.baseurl }}/images/logos/oai-codex/OAI_Codex-Lockup_400px.svg" alt="OpenAI Codex" style="max-width: 100px; max-height: 40px; object-fit: contain;"></a>
  <a href="https://block.github.io/goose/" title="Goose"><img src="{{ site.baseurl }}/images/logos/goose/goose-logo-black.png" alt="Goose" style="max-width: 100px; max-height: 40px; object-fit: contain;"></a>
  <a href="https://www.all-hands.dev/" title="OpenHands"><img src="{{ site.baseurl }}/images/logos/openhands/openhands-logo-light.svg" alt="OpenHands" style="max-width: 100px; max-height: 40px; object-fit: contain;"></a>
  <a href="https://ampcode.com/" title="Amp"><img src="{{ site.baseurl }}/images/logos/amp/amp-logo-light.svg" alt="Amp" style="max-width: 100px; max-height: 40px; object-fit: contain;"></a>
  <a href="https://databricks.com/" title="Databricks"><img src="{{ site.baseurl }}/images/logos/databricks/databricks-logo-light.svg" alt="Databricks" style="max-width: 100px; max-height: 40px; object-fit: contain;"></a>
</div>

## 开放开发

Agent Skills 格式最初由 [Anthropic](https://www.anthropic.com/) 开发，作为开放标准发布，已被越来越多的代理产品采用。该标准开放接受更广泛生态系统的贡献。

[在 GitHub 上查看](https://github.com/agentskills/agentskills)

## 开始使用

- [什么是技能？]({{ site.baseurl }}/what-are-skills.html) - 了解技能、它们如何工作以及为什么重要。
- [规范]({{ site.baseurl }}/specification.html) - SKILL.md 文件的完整格式规范。
- [添加技能支持]({{ site.baseurl }}/client-implementation/adding-skills-support.html) - 为您的代理或工具添加技能支持。
- [示例技能](https://github.com/anthropics/skills) - 在 GitHub 上浏览示例技能。
- [参考库](https://github.com/agentskills/agentskills/tree/main/skills-ref) - 验证技能并生成提示词 XML。