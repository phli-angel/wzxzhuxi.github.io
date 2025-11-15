# Jekyll 站点

## 项目简介
本仓库托管一个基于 Jekyll 的静态网站。`index.html`、`about.md`、`team.md` 等页面和 `_posts/` 中按日期命名的文章会通过 `_layouts/` 的布局与 `_includes/` 的片段渲染。样式集中在 `assets/css/style.scss`，站点元数据、导航和插件配置写在 `_config.yml`。构建输出位于 `_site/`，该目录由构建生成，不需要手动修改或提交。

## 环境准备
1. 安装 Ruby 3.x、Bundler，并保证系统中可用 `node`（供前端工具使用）。
2. 执行 `bundle install` 安装 `Gemfile` 中声明的依赖。
3. 启动开发服务器：
   ```bash
   bundle exec jekyll serve --livereload
   ```
   浏览器访问 `http://127.0.0.1:4000` 进行预览。

## 开发流程
- **内容编辑**：在 `_posts/` 中新增 `YYYY-MM-DD-title.md` 命名的 Markdown 文件，YAML 头部须包含 `layout`、`title` 及模板所需的自定义变量。
- **样式调整**：修改 `assets/css/style.scss`，该文件支持 SCSS 嵌套，Jekyll 构建时会自动编译。
- **配置修改**：变更导航、URL 结构或插件时编辑 `_config.yml`，修改后需重启 `jekyll serve`。
- **校验**：提交前运行 `bundle exec jekyll build --trace`，并使用 `bundle exec jekyll doctor` 检查失效链接或缺失资源。

## 部署
当前部署目标是 GitHub Pages（`https://wzxzhuxi.github.io/`）。通过 GitHub Actions 或 Pages 分支执行 `JEKYLL_ENV=production bundle exec jekyll build`，将 `_site/` 发布到该站点。不要把 `_site/` 提交到版本库。

## 贡献指南
遵循 `AGENTS.md` 中的分支、测试和 PR 规范。保持小而聚焦的提交，使用祈使句式（如 `Add hero section layout`），视觉改动附上截图或 GIF。提交评审前确认 `jekyll build` 与 `jekyll doctor` 均通过。
