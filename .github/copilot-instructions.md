# Copilot 指令 — umiri-blog

目的：快速让 AI 编码代理在本仓库中立即可用并高效工作。

要点摘要
- 项目类型：Hugo 静态站点（主题驱动）。主要源代码在 `content/`, `layouts/`, `assets/`, `themes/`。
- 配置文件：以 `config.toml` 为主（仓库根目录）。优先信任 `config.toml` 中的设置（例如 `theme`、`baseURL`）。

运行与发布（具体命令示例）
- 本地预览：在仓库根运行 `hugo server -D`（或 `cd umiri-blog && hugo server -D`），访问 `http://localhost:1313`。
- 生产构建：`hugo --minify`。
- 部署：由 `.github/workflows/hugo.yml` 的 GitHub Actions 自动构建并发布到 GitHub Pages；推送到主分支会触发部署。

项目结构与重要位置（示例）
- 文章内容：`content/posts/`（按 README 约定写 front matter，示例：包含 `title`, `date`, `draft`, `tags`）。
- 站点配置：`config.toml`（站点元信息、主题、分页、params 等）。
- 主题与覆盖：`themes/` 包含下游主题（不要直接修改主题内的第三方代码，优先通过 `layouts/` 覆盖）。
- 模板：`layouts/_default/`, `layouts/partials/`（放置模板覆盖或自定义模板）。
- 静态资源：`static/`（直接复制到网站根），`assets/`（用于 Hugo Pipes 处理的资源）。
- 生成产物：`public/`（已生成的静态站点，通常不应由 AI 自动修改，除非你在做部署相关更改）。

项目特有约定与注意事项
- README 中提到 `PaperMod`，但 `config.toml` 中 `theme = "blowfish"`：以 `config.toml` 为准，发现此类不一致时请在变更前指出并询问。
- 写文章时请使用仓库现有的 front-matter 风格（README 示例使用 YAML 风格 front-matter）；保持 `draft: false` 以便发布。
- 主题通常通过 submodule 或直接放在 `themes/`，若需要改动主题样式，优先在 `assets/css/extended/` 或 `layouts/` 中覆盖。
- 避免提交生成的 `public/` 内容作为手工更改；若确实需要更新，请说明原因并先征得人工确认。

样例工作流（AI 修改建议）
1. 修改 `content/` 或 `layouts/`（以源文件为准）。
2. 在本地运行 `hugo server -D` 验证页面渲染（如果环境允许）。
3. 提交变更，commit message 推荐使用语义前缀，例如 `fix:`, `feat:`, `add:`。

查阅点
- 站点配置：`config.toml`
- 自动构建：`.github/workflows/hugo.yml`
- 文章目录：`content/posts/`

当你不确定时
- 检查 `config.toml` 的 `baseURL`、`theme` 与 `params`。
- 在对主题第三方代码进行改动前，先创建覆盖模板或询问仓库所有者。

请审阅此文件并指出需要补充或修正的项目级约定或偏好。

Blowfish 主题专用实践

- 最低要求：Blowfish 要求较新的 Hugo（主题 README 标注最小版本 `0.141.0`）；此外主题构建样式需要 `Node.js` 与 `npm`（用于 Tailwind CLI 与 vendor-copy）。
- 主题配置：Blowfish 在 `themes/blowfish/config/_default/` 下提供一组推荐的 `*.toml`（如 `hugo.toml`, `params.toml`, `menus.en.toml` 等）。常见做法是将这些文件复制到站点的 `config/_default/`，或使用 Hugo Modules 导入（参考主题 README 的 "Quick start using Hugo" 部分）。

- 资产与编译：要编译或热重载 Tailwind CSS，请在主题根目录运行：

```bash
cd themes/blowfish
npm install
npm run dev      # 开发模式，watch CSS
# 或生产构建：
npm run build
```

	说明：主题 `package.json` 中的 `dev`/`build` 脚本会把编译结果输出到 `assets/css/compiled/`，并通过 `vendor-copy` 把必要的第三方库复制到 `assets/lib/`（参见 `devVendorCopy` 列表）。如果你在 CI 中需要静态构建，请在构建步骤中先运行 `npm ci` 或 `npm install`，再运行 `npm run build`，然后执行 `hugo --minify`。

- 运行示例站点：主题自带 `exampleSite`，可用于预览完整配置。示例命令：

```bash
cd themes/blowfish
npm install
npm run example   # 启动 exampleSite 的 hugo server
```

- 覆盖与自定义建议：
	- 优先通过在项目根的 `layouts/` 与 `assets/css/extended/` 添加覆盖文件来定制样式/模板，避免直接改动 `themes/blowfish` 的源文件（除非你正在维护一个 fork 或子模块）。
	- 若需要启用主题可选功能（如 Firebase、Views、Likes、KaTeX、Mermaid 等），在站点 `config/_default/params.toml` 中设置对应参数（参见 `themes/blowfish/config/_default/params.toml`）。

- CI / 部署 注意：
	- 如果使用 GitHub Actions 构建（当前仓库使用 `.github/workflows/hugo.yml`），确保在 workflow 中安装 Node.js 并运行主题的 `npm run build`（若你依赖主题编译的资产），或将编译后的静态资产纳入构建产物。
	- 使用 Hugo Modules 时，更新主题到最新可用版本：`hugo mod get -u`。

引用（仓库内片段）
- 主题配置样例：`themes/blowfish/config/_default/hugo.toml`、`themes/blowfish/config/_default/params.toml`。
- 主题脚本：`themes/blowfish/package.json`（含 `dev`, `build`, `example`, `devVendorCopy` 配置）。

如果你希望，我可以：
- 在仓库 CI (`.github/workflows/hugo.yml`) 中添加或验证 `npm install && npm run build` 步骤，或
- 在项目根创建一个示例覆盖 `layouts/_default/single.html` 并本地运行 `hugo server -D` 来展示覆盖结果。


具体示例（摘录自仓库）

- 文章 front-matter 示例（仓库中存在不同格式，请保持一致）：

	- TOML（`+++`）示例，见 [content/posts/first-post.md](content/posts/first-post.md)：

		```toml
		+++
		date = '2025-06-26T12:06:42+08:00'
		draft = 'false'
		title = 'First Post'
		tags = ["Hugo", "GitHub Pages"]
		+++
		```

	- 非标准/不一致示例（需注意）：见 [content/posts/my-first-post.md](content/posts/my-first-post.md)，该文件在仓库中以不同格式出现，建议统一为 TOML 或 YAML。

- 主题与覆盖示例：仓库包含多个主题（`themes/` 下如 `blowfish`, `PaperMod`, `ananke`）。优先使用主题提供的变量与 `config.toml` 中的 `params`，如需局部修改，创建覆盖模板：

	- 覆盖 `header` 或 `footer`：把文件放在 [layouts/partials/](layouts/partials/) 下（如果不存在，则创建）。
	- 覆盖页面模板：放在 [layouts/_default/](layouts/_default/)，例如自定义 `single.html` 或 `list.html`。

- 样式与资源：优先在 [assets/css/extended/](assets/css/extended/) 放自定义 CSS（示例目录存在但当前为空），或在 `themes/<theme>/assets/` 中查找主题样式并通过覆盖替换。

- 生成产物与 CI：查看 GitHub Actions 工作流 [.github/workflows/hugo.yml](.github/workflows/hugo.yml) 以了解部署分支与构建步骤。`public/` 目录是生成产物，通常由 CI 生成且不应手动编辑（示例： [public/index.html](public/index.html)）。

快速命令（复制粘贴运行）：

```bash
# 本地预览（草稿可见）
hugo server -D

# 生产构建
hugo --minify
```

注意：仓库 README 中示例使用 YAML 风格 front-matter，但实际内容包含 TOML 等不同风格。任何批量修改 front-matter 前先在单个示例页面测试渲染。

请确认是否需要我：
- 统一 `content/posts/` 的 front-matter 风格（我可以先在一个示例文件上演示并运行 `hugo server -D`）。
- 增加更多针对 `layouts/` 覆盖的具体代码片段或 PR 模板。
