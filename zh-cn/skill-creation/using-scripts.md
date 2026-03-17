---
title: "在技能中使用脚本"
layout: default
---

技能可以指示代理运行 shell 命令，并在 `scripts/` 目录中打包可复用脚本。本指南涵盖一次性命令、自带依赖的独立脚本，以及如何设计适合代理使用的脚本接口。

## 一次性命令

当现有的包已经满足你的需求时，你可以直接在 `SKILL.md` 指令中引用它，而不需要 `scripts/` 目录。许多生态系统提供了在运行时自动解析依赖的工具。

<details>
<summary>uvx</summary>

[uvx](https://docs.astral.sh/uv/guides/tools/) 在隔离环境中运行 Python 包，并具有激进的缓存。它随 [uv](https://docs.astral.sh/uv/) 一起发布。

```bash
uvx ruff@0.8.0 check .
uvx black@24.10.0 .
```

- 不随 Python 附带——需要单独安装。
- 快速。缓存激进，重复运行几乎即时。

</details>

<details>
<summary>pipx</summary>

[pipx](https://pipx.pypa.io/) 在隔离环境中运行 Python 包。可通过操作系统包管理器安装（`apt install pipx`、`brew install pipx`）。

```bash
pipx run 'black==24.10.0' .
pipx run 'ruff==0.8.0' check .
```

- 不随 Python 附带——需要单独安装。
- `uvx` 的成熟替代方案。虽然 `uvx` 已成为标准推荐，但 `pipx` 仍然是可靠的选择，在操作系统包管理器中有更广泛的可用性。

</details>

<details>
<summary>npx</summary>

[npx](https://docs.npmjs.com/cli/commands/npx) 运行 npm 包，按需下载。它随 npm 一起发布（npm 随 Node.js 一起发布）。

```bash
npx eslint@9 --fix .
npx create-vite@6 my-app
```

- 随 Node.js 附带——无需额外安装。
- 下载包、运行并缓存以备后用。
- 使用 `npx package@version` 固定版本以确保可重复性。

</details>

<details>
<summary>bunx</summary>

[bunx](https://bun.sh/docs/cli/bunx) 是 Bun 的 `npx` 等价物。它随 [Bun](https://bun.sh/) 一起发布。

```bash
bunx eslint@9 --fix .
bunx create-vite@6 my-app
```

- 在基于 Bun 的环境中可直接替代 `npx`。
- 只有当用户环境有 Bun 而非 Node.js 时才适用。

</details>

<details>
<summary>deno run</summary>

[deno run](https://docs.deno.com/runtime/reference/cli/run/) 直接从 URL 或说明符运行脚本。它随 [Deno](https://deno.com/) 一起发布。

```bash
deno run npm:create-vite@6 my-app
deno run --allow-read npm:eslint@9 -- --fix .
```

- 文件系统/网络访问需要权限标志（`--allow-read` 等）。
- 使用 `--` 分隔 Deno 标志和工具自己的标志。

</details>

<details>
<summary>go run</summary>

[go run](https://pkg.go.dev/cmd/go#hdr-Compile_and_run_Go_program) 直接编译并运行 Go 包。它内置于 `go` 命令中。

```bash
go run golang.org/x/tools/cmd/goimports@v0.28.0 .
go run github.com/golangci/golangci-lint/cmd/golangci-lint@v1.62.0 run
```

- 内置于 Go——无需额外工具。
- 固定版本或使用 `@latest` 使命令明确。

</details>

**在技能中使用一次性命令的技巧：**

- **固定版本**（例如 `npx eslint@9.0.0`）使命令随时间保持一致的行为。
- **在 `SKILL.md` 中说明前提条件**（例如 "Requires Node.js 18+"），而不是假设代理的环境已有它们。对于运行时级别的要求，使用 [`compatibility` 前置元数据字段]({{ site.baseurl }}/specification.html#compatibility-field)。
- **将复杂命令移入脚本。** 当你用几个标志调用工具时，一次性命令效果很好。当命令变得足够复杂以至于第一次就很难弄对时，`scripts/` 中经过测试的脚本更可靠。

## 从 `SKILL.md` 引用脚本

使用**从技能目录根目录开始的相对路径**来引用打包文件。代理自动解析这些路径——不需要绝对路径。

在 `SKILL.md` 中列出可用脚本，让代理知道它们的存在：

```markdown SKILL.md
## Available scripts

- **`scripts/validate.sh`** — 验证配置文件
- **`scripts/process.py`** — 处理输入数据
```

然后指示代理运行它们：

````markdown SKILL.md
## Workflow

1. Run the validation script:
   ```bash
   bash scripts/validate.sh "$INPUT_FILE"
   ```

2. Process the results:
   ```bash
   python3 scripts/process.py --input results.json
   ```
````

> **注意：** 同样的相对路径约定也适用于 `references/*.md` 等支持文件——脚本执行路径（在代码块中）相对于**技能目录根目录**，因为代理从那里运行命令。

## 独立脚本

当你需要可复用的逻辑时，在 `scripts/` 中打包一个内联声明依赖的脚本。代理可以用单个命令运行脚本——无需单独的清单文件或安装步骤。

几种语言支持内联依赖声明：

<details>
<summary>Python</summary>

[PEP 723](https://peps.python.org/pep-0723/) 定义了内联脚本元数据的标准格式。在 `# ///` 标记内的 TOML 块中声明依赖：

```python scripts/extract.py
# /// script
# dependencies = [
#   "beautifulsoup4",
# ]
# ///

from bs4 import BeautifulSoup

html = '<html><body><h1>Welcome</h1><p class="info">This is a test.</p></body></html>'
print(BeautifulSoup(html, "html.parser").select_one("p.info").get_text())
```

使用 [uv](https://docs.astral.sh/uv/) 运行（推荐）：

```bash
uv run scripts/extract.py
```

`uv run` 创建隔离环境、安装声明的依赖并运行脚本。[pipx](https://pipx.pypa.io/)（`pipx run scripts/extract.py`）也支持 PEP 723。

- 使用 [PEP 508](https://peps.python.org/pep-0508/) 说明符固定版本：`"beautifulsoup4>=4.12,<5"`。
- 使用 `requires-python` 约束 Python 版本。
- 使用 `uv lock --script` 创建锁定文件以实现完全可重复性。

</details>

<details>
<summary>Deno</summary>

Deno 的 `npm:` 和 `jsr:` 导入说明符使每个脚本默认独立：

```typescript scripts/extract.ts
#!/usr/bin/env -S deno run

import * as cheerio from "npm:cheerio@1.0.0";

const html = `<html><body><h1>Welcome</h1><p class="info">This is a test.</p></body></html>`;
const $ = cheerio.load(html);
console.log($("p.info").text());
```

```bash
deno run scripts/extract.ts
```

- 使用 `npm:` 表示 npm 包，`jsr:` 表示 Deno 原生包。
- 版本说明符遵循 semver：`@1.0.0`（精确）、`@^1.0.0`（兼容）。
- 依赖全局缓存。使用 `--reload` 强制重新获取。
- 带原生插件（node-gyp）的包可能不工作——发布预编译二进制文件的包效果最好。

</details>

<details>
<summary>Bun</summary>

当没有找到 `node_modules` 目录时，Bun 在运行时自动安装缺失的包。直接在导入路径中固定版本：

```typescript scripts/extract.ts
#!/usr/bin/env bun

import * as cheerio from "cheerio@1.0.0";

const html = `<html><body><h1>Welcome</h1><p class="info">This is a test.</p></body></html>`;
const $ = cheerio.load(html);
console.log($("p.info").text());
```

```bash
bun run scripts/extract.ts
```

- 不需要 `package.json` 或 `node_modules`。TypeScript 原生工作。
- 包全局缓存。首次运行下载；后续运行几乎即时。
- 如果目录树中任何位置存在 `node_modules` 目录，自动安装将被禁用，Bun 会回退到标准 Node.js 解析。

</details>

<details>
<summary>Ruby</summary>

Bundler 自 Ruby 2.6 起随附。使用 `bundler/inline` 直接在脚本中声明 gem：

```ruby scripts/extract.rb
require 'bundler/inline'

gemfile do
  source 'https://rubygems.org'
  gem 'nokogiri'
end

html = '<html><body><h1>Welcome</h1><p class="info">This is a test.</p></body></html>'
doc = Nokogiri::HTML(html)
puts doc.at_css('p.info').text
```

```bash
ruby scripts/extract.rb
```

- 显式固定版本（`gem 'nokogiri', '~> 1.16'`）——没有锁定文件。
- 工作目录中现有的 `Gemfile` 或 `BUNDLE_GEMFILE` 环境变量可能会干扰。

</details>

## 为代理使用设计脚本

当代理运行你的脚本时，它会读取 stdout 和 stderr 来决定下一步做什么。一些设计选择可以使脚本对代理来说更容易使用。

### 避免交互式提示

这是代理执行环境的硬性要求。代理在非交互式 shell 中运行——它们无法响应 TTY 提示、密码对话框或确认菜单。阻塞等待交互式输入的脚本将无限期挂起。

通过命令行标志、环境变量或 stdin 接受所有输入：

```
# 错误：阻塞等待输入
$ python scripts/deploy.py
Target environment: _

# 正确：带指导的清晰错误
$ python scripts/deploy.py
Error: --env is required. Options: development, staging, production.
Usage: python scripts/deploy.py --env staging --tag v1.2.3
```

### 用 `--help` 记录用法

`--help` 输出是代理了解脚本接口的主要方式。包含简要描述、可用标志和使用示例：

```
Usage: scripts/process.py [OPTIONS] INPUT_FILE

Process input data and produce a summary report.

Options:
  --format FORMAT    Output format: json, csv, table (default: json)
  --output FILE      Write output to FILE instead of stdout
  --verbose          Print progress to stderr

Examples:
  scripts/process.py data.csv
  scripts/process.py --format csv --output report.csv data.csv
```

保持简洁——输出会进入代理的上下文窗口，与它处理的所有其他内容并存。

### 编写有帮助的错误消息

当代理遇到错误时，消息直接影响它的下一次尝试。模糊的"Error: invalid input"会浪费一轮对话。相反，说明哪里出了问题、预期是什么、应该尝试什么：

```
Error: --format must be one of: json, csv, table.
       Received: "xml"
```

### 使用结构化输出

优先使用结构化格式——JSON、CSV、TSV——而非自由格式文本。结构化格式可以被代理和标准工具（`jq`、`cut`、`awk`）使用，使你的脚本可以在管道中组合。

```
# 空格对齐——难以程序化解析
NAME          STATUS    CREATED
my-service    running   2025-01-15

# 分隔——字段边界明确
{"name": "my-service", "status": "running", "created": "2025-01-15"}
```

**将数据与诊断分离：** 将结构化数据发送到 stdout，将进度消息、警告和其他诊断信息发送到 stderr。这让代理可以捕获干净、可解析的输出，同时在需要时仍能访问诊断信息。

### 其他注意事项

- **幂等性。** 代理可能会重试命令。"不存在则创建"比"创建并在重复时失败"更安全。
- **输入约束。** 用清晰的错误拒绝歧义输入，而不是猜测。在可能的情况下使用枚举和封闭集合。
- **试运行支持。** 对于破坏性或有状态操作，`--dry-run` 标志让代理可以预览将要发生什么。
- **有意义的退出码。** 为不同的失败类型使用不同的退出码（未找到、参数无效、认证失败）并在 `--help` 输出中记录它们，让代理知道每个代码的含义。
- **安全的默认值。** 考虑破坏性操作是否需要显式确认标志（`--confirm`、`--force`）或其他适合风险级别的保护措施。
- **可预测的输出大小。** 许多代理框架会自动截断超过阈值（例如 10-30K 字符）的工具输出，可能丢失关键信息。如果你的脚本可能产生大量输出，默认使用摘要或合理限制，并支持像 `--offset` 这样的标志，让代理可以在需要时请求更多信息。或者，如果输出很大且不适合分页，要求代理传递 `--output` 标志指定输出文件或 `-` 以显式选择输出到 stdout。