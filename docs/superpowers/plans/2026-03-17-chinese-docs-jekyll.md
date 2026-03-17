# Agent Skills 中文文档站实现计划

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 创建中文文档站，使用 Jekyll + GitHub Pages 托管，多子代理并行翻译

**Architecture:** 独立 zh-cn/ 目录，Jekyll minima 主题，GitHub Actions 自动部署，4 个子代理并行翻译 8 个文档文件

**Tech Stack:** Jekyll, GitHub Pages, GitHub Actions, Markdown

---

## 文件结构

| 文件 | 职责 | 操作 |
|------|------|------|
| `zh-cn/_config.yml` | Jekyll 主配置文件 | 创建 |
| `zh-cn/_layouts/default.html` | 默认页面布局模板 | 创建 |
| `zh-cn/_includes/nav.html` | 导航组件 | 创建 |
| `zh-cn/_data/navigation.yml` | 导航配置 | 创建 |
| `zh-cn/assets/style.css` | 自定义样式 | 创建 |
| `zh-cn/assets/favicon.svg` | 网站图标 | 复制 |
| `zh-cn/images/logos/` | Logo 图片资源 | 复制 |
| `zh-cn/index.md` | 首页（翻译自 home.mdx） | 翻译 |
| `zh-cn/what-are-skills.md` | 什么是技能 | 翻译 |
| `zh-cn/specification.md` | 规范 | 翻译 |
| `zh-cn/skill-creation/best-practices.md` | 最佳实践 | 翻译 |
| `zh-cn/skill-creation/optimizing-descriptions.md` | 优化描述 | 翻译 |
| `zh-cn/skill-creation/evaluating-skills.md` | 评估技能 | 翻译 |
| `zh-cn/skill-creation/using-scripts.md` | 使用脚本 | 翻译 |
| `zh-cn/client-implementation/adding-skills-support.md` | 添加技能支持 | 翻译 |
| `.github/workflows/jekyll.yml` | GitHub Actions 部署配置 | 创建 |

---

## Chunk 1: Jekyll 框架搭建

### Task 1: 创建目录结构

**Files:**
- Create: `zh-cn/` 目录及子目录

- [ ] **Step 1: 创建 zh-cn 目录结构**

```bash
mkdir -p zh-cn/_layouts
mkdir -p zh-cn/_includes
mkdir -p zh-cn/_data
mkdir -p zh-cn/assets
mkdir -p zh-cn/images/logos
mkdir -p zh-cn/skill-creation
mkdir -p zh-cn/client-implementation
```

- [ ] **Step 2: 验证目录创建成功**

Run: `ls -la zh-cn/`
Expected: 显示 _layouts, _includes, _data, assets, images, skill-creation, client-implementation 目录

---

### Task 2: 创建 Jekyll 配置文件

**Files:**
- Create: `zh-cn/_config.yml`

- [ ] **Step 1: 创建 _config.yml**

```yaml
title: Agent Skills 中文文档
description: 一种简单、开放的格式，用于赋予 AI 代理新能力和专业知识
lang: zh-CN
url: ""
baseurl: "/agentskills_cn"

theme: minima

header_pages:
  - what-are-skills.md
  - specification.md

markdown: kramdown
kramdown:
  input: GFM
  syntax_highlighter: rouge

defaults:
  - scope:
      path: ""
      type: "pages"
    values:
      layout: default

exclude:
  - README.md
  - Gemfile
  - Gemfile.lock
```

- [ ] **Step 2: 验证配置文件语法**

Run: `cat zh-cn/_config.yml`
Expected: 显示完整的 YAML 配置内容

---

### Task 3: 创建默认布局模板

**Files:**
- Create: `zh-cn/_layouts/default.html`

- [ ] **Step 1: 创建 default.html 布局**

```html
<!DOCTYPE html>
<html lang="{{ page.lang | default: site.lang | default: 'zh-CN' }}">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>{% if page.title %}{{ page.title }} | {% endif %}{{ site.title }}</title>
  <meta name="description" content="{{ page.description | default: site.description }}">
  <link rel="stylesheet" href="{{ site.baseurl }}/assets/style.css">
  <link rel="icon" href="{{ site.baseurl }}/assets/favicon.svg" type="image/svg+xml">
</head>
<body>
  <header class="site-header">
    <div class="wrapper">
      <a class="site-title" href="{{ site.baseurl }}/">{{ site.title }}</a>
      {% include nav.html %}
    </div>
  </header>
  <main class="page-content">
    <div class="wrapper">
      <article class="post">
        <header class="post-header">
          <h1 class="post-title">{{ page.title }}</h1>
        </header>
        <div class="post-content">
          {{ content }}
        </div>
      </article>
    </div>
  </main>
  <footer class="site-footer">
    <div class="wrapper">
      <p>
        <a href="https://github.com/agentskills/agentskills">GitHub</a> |
        <a href="https://agentskills.io">English</a>
      </p>
    </div>
  </footer>
</body>
</html>
```

- [ ] **Step 2: 验证布局文件创建成功**

Run: `cat zh-cn/_layouts/default.html | head -5`
Expected: 显示 `<!DOCTYPE html>` 开头的内容

---

### Task 4: 创建导航组件

**Files:**
- Create: `zh-cn/_includes/nav.html`
- Create: `zh-cn/_data/navigation.yml`

- [ ] **Step 1: 创建导航数据文件**

```yaml
# zh-cn/_data/navigation.yml
main:
  - title: "什么是技能"
    url: /what-are-skills.html
  - title: "规范"
    url: /specification.html

creators:
  - title: "最佳实践"
    url: /skill-creation/best-practices.html
  - title: "优化描述"
    url: /skill-creation/optimizing-descriptions.html
  - title: "评估技能"
    url: /skill-creation/evaluating-skills.html
  - title: "使用脚本"
    url: /skill-creation/using-scripts.html

clients:
  - title: "添加技能支持"
    url: /client-implementation/adding-skills-support.html
```

- [ ] **Step 2: 创建导航 HTML 组件**

```html
<!-- zh-cn/_includes/nav.html -->
<nav class="site-nav">
  <ul class="nav-list">
    {% for item in site.data.navigation.main %}
    <li><a href="{{ site.baseurl }}{{ item.url }}">{{ item.title }}</a></li>
    {% endfor %}
    <li class="nav-group">
      <span>技能创建者</span>
      <ul>
        {% for item in site.data.navigation.creators %}
        <li><a href="{{ site.baseurl }}{{ item.url }}">{{ item.title }}</a></li>
        {% endfor %}
      </ul>
    </li>
    <li class="nav-group">
      <span>客户端实现</span>
      <ul>
        {% for item in site.data.navigation.clients %}
        <li><a href="{{ site.baseurl }}{{ item.url }}">{{ item.title }}</a></li>
        {% endfor %}
      </ul>
    </li>
  </ul>
</nav>
```

- [ ] **Step 3: 验证文件创建成功**

Run: `ls zh-cn/_includes/ zh-cn/_data/`
Expected: 显示 nav.html 和 navigation.yml

---

### Task 5: 创建样式文件

**Files:**
- Create: `zh-cn/assets/style.css`

- [ ] **Step 1: 创建 style.css**

```css
/* zh-cn/assets/style.css */
:root {
  --primary-color: #7f7f7f;
  --text-color: #333;
  --bg-color: #fff;
  --border-color: #e1e4e8;
}

* {
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  line-height: 1.6;
  color: var(--text-color);
  background-color: var(--bg-color);
  margin: 0;
  padding: 0;
}

.wrapper {
  max-width: 960px;
  margin: 0 auto;
  padding: 0 20px;
}

/* Header */
.site-header {
  border-bottom: 1px solid var(--border-color);
  padding: 15px 0;
}

.site-title {
  font-size: 1.5rem;
  font-weight: 600;
  color: var(--text-color);
  text-decoration: none;
}

/* Navigation */
.site-nav {
  float: right;
}

.nav-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.nav-list > li {
  display: inline-block;
  margin-left: 20px;
  position: relative;
}

.nav-list a {
  color: var(--primary-color);
  text-decoration: none;
}

.nav-list a:hover {
  text-decoration: underline;
}

.nav-group > ul {
  display: none;
  position: absolute;
  top: 100%;
  left: 0;
  background: var(--bg-color);
  border: 1px solid var(--border-color);
  padding: 10px;
  min-width: 150px;
  z-index: 100;
}

.nav-group:hover > ul {
  display: block;
}

.nav-group > span {
  cursor: pointer;
  color: var(--primary-color);
}

/* Main content */
.page-content {
  padding: 40px 0;
}

.post-title {
  font-size: 2rem;
  margin-bottom: 20px;
}

.post-content h2 {
  margin-top: 2rem;
  padding-bottom: 0.3rem;
  border-bottom: 1px solid var(--border-color);
}

.post-content h3 {
  margin-top: 1.5rem;
}

.post-content pre {
  background: #f6f8fa;
  padding: 16px;
  border-radius: 6px;
  overflow-x: auto;
}

.post-content code {
  background: #f6f8fa;
  padding: 0.2em 0.4em;
  border-radius: 3px;
}

.post-content pre code {
  background: none;
  padding: 0;
}

.post-content blockquote {
  border-left: 4px solid var(--primary-color);
  padding-left: 1rem;
  margin-left: 0;
  color: #666;
}

.post-content table {
  border-collapse: collapse;
  width: 100%;
  margin: 1rem 0;
}

.post-content th,
.post-content td {
  border: 1px solid var(--border-color);
  padding: 8px 12px;
}

.post-content th {
  background: #f6f8fa;
}

/* Footer */
.site-footer {
  border-top: 1px solid var(--border-color);
  padding: 20px 0;
  text-align: center;
  color: #666;
}

.site-footer a {
  color: var(--primary-color);
}

/* Logo grid for home page */
.logo-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
  gap: 20px;
  margin: 2rem 0;
}

.logo-grid img {
  max-width: 100px;
  max-height: 50px;
  object-fit: contain;
}

/* Cards */
.card-group {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 20px;
  margin: 2rem 0;
}

.card {
  border: 1px solid var(--border-color);
  border-radius: 8px;
  padding: 20px;
}

.card h3 {
  margin-top: 0;
}

.card a {
  color: var(--primary-color);
}

/* Responsive */
@media (max-width: 600px) {
  .site-nav {
    float: none;
    margin-top: 10px;
  }

  .nav-list > li {
    display: block;
    margin-left: 0;
    margin-top: 5px;
  }
}
```

- [ ] **Step 2: 验证样式文件创建成功**

Run: `wc -l zh-cn/assets/style.css`
Expected: 显示行数 > 100

---

### Task 6: 创建 GitHub Actions 工作流

**Files:**
- Create: `.github/workflows/jekyll.yml`

- [ ] **Step 1: 创建 .github/workflows 目录**

```bash
mkdir -p .github/workflows
```

- [ ] **Step 2: 创建 jekyll.yml 工作流文件**

```yaml
name: Deploy Jekyll site to Pages

on:
  push:
    branches: ["main"]
    paths: ["zh-cn/**"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Setup Pages
        uses: actions/configure-pages@v4
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: "zh-cn"
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

- [ ] **Step 3: 验证工作流文件创建成功**

Run: `cat .github/workflows/jekyll.yml | head -10`
Expected: 显示 `name: Deploy Jekyll site to Pages`

- [ ] **Step 4: 提交 Chunk 1 更改**

```bash
git add zh-cn/ .github/workflows/jekyll.yml
git commit -m "feat: setup Jekyll framework for Chinese docs

- Create zh-cn directory structure
- Add _config.yml with minima theme
- Add default layout and navigation components
- Add custom CSS styles
- Add GitHub Actions workflow for deployment"
```

---

## Chunk 2: 资源复制

### Task 7: 复制 favicon 和 logo 图片

**Files:**
- Copy: `docs/favicon.svg` → `zh-cn/assets/favicon.svg`
- Copy: `docs/images/logos/*` → `zh-cn/images/logos/`

- [ ] **Step 1: 复制 favicon.svg**

```bash
cp docs/favicon.svg zh-cn/assets/favicon.svg
```

- [ ] **Step 2: 复制 logo 图片**

```bash
cp -r docs/images/logos/* zh-cn/images/logos/
```

- [ ] **Step 3: 验证复制成功**

Run: `ls zh-cn/assets/favicon.svg zh-cn/images/logos/ | head -5`
Expected: 显示 favicon.svg 和多个 logo 文件

- [ ] **Step 4: 提交资源文件**

```bash
git add zh-cn/assets/favicon.svg zh-cn/images/logos/
git commit -m "chore: copy favicon and logo assets for Chinese docs"
```

---

## Chunk 3: 并行翻译文档

### Task 8: 启动 4 个子代理并行翻译

**翻译规范参考:** `docs/superpowers/specs/2026-03-17-chinese-docs-jekyll-design.md`

**子代理任务分配:**

| 子代理 | 输入文件 | 输出文件 |
|--------|----------|----------|
| Agent 1 | docs/home.mdx, docs/what-are-skills.mdx | zh-cn/index.md, zh-cn/what-are-skills.md |
| Agent 2 | docs/specification.mdx | zh-cn/specification.md |
| Agent 3 | docs/skill-creation/best-practices.mdx, docs/skill-creation/optimizing-descriptions.mdx | zh-cn/skill-creation/best-practices.md, zh-cn/skill-creation/optimizing-descriptions.md |
| Agent 4 | docs/skill-creation/evaluating-skills.mdx, docs/skill-creation/using-scripts.mdx, docs/client-implementation/adding-skills-support.mdx | zh-cn/skill-creation/evaluating-skills.md, zh-cn/skill-creation/using-scripts.md, zh-cn/client-implementation/adding-skills-support.md |

- [ ] **Step 1: 使用 subagent-driven-development 技能启动并行翻译**

翻译时每个子代理需要遵循以下规范：

**术语对照表:**
| 英文 | 中文 |
|------|------|
| Agent Skills | Agent Skills |
| skill | 技能 |
| SKILL.md | SKILL.md |
| agent | 代理 |
| frontmatter | 前置元数据 |
| progressive disclosure | 渐进式披露 |
| prompt | 提示词 |
| context | 上下文 |
| client | 客户端 |
| creator | 创建者 |
| metadata | 元数据 |
| specification | 规范 |

**组件转换:**
- `<Tip>内容</Tip>` → `> **提示：** 内容`
- `<Note>内容</Note>` → `> **注意：** 内容`
- `<Card>` → 列表项或自定义 HTML
- `<CardGroup>` → `<div class="card-group">...</div>`

**链接映射:**
- `/what-are-skills` → `/what-are-skills.html`
- `/specification` → `/specification.html`
- `/skill-creation/best-practices` → `/skill-creation/best-practices.html`
- 等等...

**Frontmatter 格式:**
```yaml
---
title: "中文标题"
description: "中文描述"
---
```

---

### Task 9: 验证翻译结果

- [ ] **Step 1: 检查所有翻译文件是否存在**

```bash
ls -la zh-cn/*.md zh-cn/skill-creation/*.md zh-cn/client-implementation/*.md
```

Expected: 显示 8 个 .md 文件

- [ ] **Step 2: 检查文件内容是否有中文**

Run: `grep -l "技能\|代理\|规范" zh-cn/*.md`
Expected: 显示至少 2 个文件

- [ ] **Step 3: 检查链接格式是否正确**

Run: `grep -E '\]\(/[^h]' zh-cn/**/*.md | head -5`
Expected: 显示内部链接，应以 `.html` 结尾

---

### Task 10: 最终提交和部署验证

- [ ] **Step 1: 提交所有翻译文件**

```bash
git add zh-cn/
git commit -m "feat: translate Agent Skills docs to Chinese

- Translate home.mdx to index.md
- Translate what-are-skills.mdx
- Translate specification.mdx
- Translate skill-creation guides
- Translate client-implementation guide

Co-Authored-By: Claude <noreply@anthropic.com>"
```

- [ ] **Step 2: 推送到远程仓库**

```bash
git push origin main
```

- [ ] **Step 3: 检查 GitHub Actions 是否触发**

Run: `gh run list --limit 1`
Expected: 显示最新的工作流运行

- [ ] **Step 4: 等待部署完成后验证站点**

访问 `https://<username>.github.io/agentskills_cn/` 验证中文文档站是否正常显示。