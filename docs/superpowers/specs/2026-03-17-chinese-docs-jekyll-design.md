# Agent Skills 中文文档站设计

## 概述

将 Agent Skills 文档翻译为简体中文，并使用 GitHub Pages (Jekyll) 托管中文站，与原有 Mintlify 英文站并存。

## 目标

- 提供完整的中文文档站
- 保留原英文站点不变
- 支持多子代理并行翻译以加快进度
- 使用 GitHub Pages 原生支持的 Jekyll，降低维护成本

## 目录结构

```
agentskills_cn/
├── docs/                    # 保留原有 Mintlify 英文文档
│   ├── docs.json
│   ├── home.mdx
│   └── ...
├── zh-cn/                   # 新建中文文档目录
│   ├── _config.yml          # Jekyll 配置
│   ├── _layouts/            # 页面布局模板
│   │   └── default.html
│   ├── _includes/           # 可复用组件
│   ├── assets/              # 样式和图片
│   │   └── style.css
│   ├── index.md             # 首页
│   ├── what-are-skills.md   # 翻译后的文档
│   ├── specification.md
│   └── skill-creation/
│       ├── best-practices.md
│       ├── optimizing-descriptions.md
│       ├── evaluating-skills.md
│       └── using-scripts.md
│   └── client-implementation/
│       └── adding-skills-support.md
└── .github/
    └── workflows/
        └── jekyll.yml       # GitHub Actions 部署配置
```

## 技术选型

### 静态站点生成器
- **Jekyll** - GitHub Pages 原生支持，无需额外构建步骤
- **主题**: minima - 简洁轻量，适合文档站

### 部署方式
- **GitHub Pages** - 自动部署，免费托管
- **GitHub Actions** - 推送到 main 分支时自动构建

## Jekyll 配置

### _config.yml

```yaml
title: Agent Skills 中文文档
description: 一种简单、开放的格式，用于赋予 AI 代理新能力和专业知识
lang: zh-CN
url: "https://<username>.github.io"
baseurl: "/agentskills"

theme: minima

header_pages:
  - what-are-skills.md
  - specification.md
  - skill-creation/best-practices.md

markdown: kramdown
kramdown:
  input: GFM
```

### GitHub Actions 工作流

```yaml
name: Deploy Jekyll site to Pages

on:
  push:
    branches: ["main"]
    paths: ["zh-cn/**"]

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/configure-pages@v4
      - uses: actions/upload-pages-artifact@v3
        with:
          path: "zh-cn"
      - uses: actions/deploy-pages@v4
```

## 多子代理并行翻译策略

### 文件分配

| 子代理 | 负责文件 | 原文件路径 |
|--------|----------|-----------|
| Agent 1 | home.mdx, what-are-skills.mdx | docs/home.mdx, docs/what-are-skills.mdx |
| Agent 2 | specification.mdx | docs/specification.mdx |
| Agent 3 | best-practices.mdx, optimizing-descriptions.mdx | docs/skill-creation/best-practices.mdx, docs/skill-creation/optimizing-descriptions.mdx |
| Agent 4 | evaluating-skills.mdx, using-scripts.mdx, adding-skills-support.mdx | docs/skill-creation/evaluating-skills.mdx, docs/skill-creation/using-scripts.mdx, docs/client-implementation/adding-skills-support.mdx |

### 术语对照表

| 英文 | 中文 | 备注 |
|------|------|------|
| Agent Skills | Agent Skills | 不翻译，品牌名称 |
| skill | 技能 | |
| SKILL.md | SKILL.md | 不翻译，文件名 |
| agent | 代理 | |
| frontmatter | 前置元数据 | |
| progressive disclosure | 渐进式披露 | |
| prompt | 提示词 | |
| context | 上下文 | |
| client | 客户端 | |
| creator | 创建者 | |
| metadata | 元数据 | |
| specification | 规范 | |

### 翻译要求

1. **保留不变的内容**：
   - 代码块内容
   - 文件路径
   - URL 链接
   - YAML frontmatter 字段名（name, description 等）

2. **需要翻译的内容**：
   - 所有正文文本
   - 标题和副标题
   - frontmatter 中的值（title, description 的值）
   - 表格内容

3. **格式转换**：
   - .mdx → .md（移除 JSX 组件或转换为 HTML）
   - Mintlify 组件（Card, Tip 等）转换为 Markdown 或 HTML 等效形式

## 实现步骤

### 阶段 1：搭建 Jekyll 框架
1. 创建 `zh-cn/` 目录结构
2. 编写 `_config.yml`
3. 创建 `_layouts/default.html` 布局模板
4. 创建 `assets/style.css` 样式文件
5. 配置 `.github/workflows/jekyll.yml`

### 阶段 2：并行翻译文档
1. 同时启动 4 个子代理
2. 每个子代理读取分配的源文件
3. 按术语对照表翻译
4. 输出 .md 文件到 zh-cn/ 对应目录

### 阶段 3：整合与验证
1. 检查术语一致性
2. 验证内部链接正确性
3. 本地 Jekyll 预览测试
4. 提交代码并推送
5. 验证 GitHub Pages 部署成功

## 预期产出

- **中文站 URL**: `https://<username>.github.io/agentskills/`
- **英文站**: 保持原有 Mintlify 部署不变
- **维护方式**: 更新 zh-cn/ 目录下的 .md 文件即可更新中文文档