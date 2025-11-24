# 投稿与协作指南

欢迎为 **A_Bit_S Blog** 贡献内容！

## 投稿方式

### 方式一：GitHub Pull Request（推荐）

**适合**：熟悉 Git 的贡献者

```bash
# 1. Fork 仓库
# 在 GitHub 上点击 Fork 按钮

# 2. Clone 到本地
git clone https://github.com/YOUR_USERNAME/wzxzhuxi.github.io.git
cd wzxzhuxi.github.io

# 3. 配置上游仓库
git remote add upstream https://github.com/wzxzhuxi/wzxzhuxi.github.io.git

# 4. 创建功能分支
git checkout -b add-my-article

# 5. 添加文章（参考 _posts/POST_TEMPLATE.md 模板）
# 文件放入 _posts/YYYY/ 目录
nano _posts/2025/2025-01-18-article-title.md

# 6. 提交（参考下方 Commit 规范）
git add .
git commit -m "[feat] 添加文章标题"

# 7. 推送到你的 Fork
git push origin add-my-article

# 8. 在 GitHub 创建 Pull Request
```

### 方式二：直接协作（核心团队）

**适合**：被添加为 Collaborator 的核心成员

```bash
# 1. Clone 原始仓库
git clone https://github.com/wzxzhuxi/wzxzhuxi.github.io.git
cd wzxzhuxi.github.io

# 2. 创建功能分支
git checkout -b add-my-article

# 3. 添加内容（参考 _posts/POST_TEMPLATE.md 模板）
nano _posts/2025/2025-01-18-my-article.md

# 4. 提交并推送
git add .
git commit -m "[feat] 添加我的文章"
git push origin add-my-article

# 5. 创建 Pull Request（推荐）
```

### 方式三：GitHub Issue 投稿

**适合**：不熟悉 Git 的作者

1. 访问 [Issues 页面](https://github.com/wzxzhuxi/wzxzhuxi.github.io/issues/new)
2. 标题：`[投稿] 文章标题`
3. 粘贴 Markdown 内容（包含 front matter）
4. 上传图片或提供链接
5. 提交 Issue

### 方式四：邮件投稿

**适合**：完全不熟悉 GitHub 的作者

发送 Markdown 文件到指定邮箱（w1355457260@gmail.com）

---

## Commit 信息规范

### 格式

```
[类型] 简短描述（不超过 50 字）

- 详细说明第一点
- 详细说明第二点
```

### 类型标签

- `[feat]` - 新增内容（文章、书籍、功能）
- `[fix]` - 修复错误（链接、格式、拼写）
- `[docs]` - 文档更新
- `[style]` - 样式调整（CSS、布局）
- `[refactor]` - 重构
- `[chore]` - 杂项（依赖更新、配置）

### 示例

```bash
# 简单提交
git commit -m "[feat] 添加 Rust 所有权详解"

# 详细提交
git commit -m "[feat] 添加 Python 基础教程系列

- 新增书籍元数据和 5 个章节
- 添加配套代码示例和练习题
- 更新书籍索引页面"
```

---

## 内容格式规范

详细规范请参考 [CLAUDE.md](./CLAUDE.md)

### 文章格式（`_posts/YYYY/YYYY-MM-DD-slug.md`）

**参考模板**: [`_posts/POST_TEMPLATE.md`](./_posts/POST_TEMPLATE.md)

```yaml
---
layout: post
title: "文章标题"
author: author_key  # 必须使用 _data/authors.yml 中定义的 key (如 zhuxi, phli, lele)
date: YYYY-MM-DD
categories: [分类名]  # 从预定义分类选择,见下方说明
tags: [标签1, 标签2]  # 技术名称标签
excerpt: 1-2 句话的摘要,显示在列表页和社交分享
cover: /assets/images/posts/YYYY-MM-DD-slug/cover.png  # 可选
---

## 第一个标题

从 `##` 开始，layout 已提供 `<h1>`。

\`\`\`cpp
// 代码示例必须标注语言
int main() { return 0; }
\`\`\`

![图片描述](/assets/images/posts/YYYY-MM-DD-slug/diagram-example.png)
*可选的图片说明文字*
```

#### 预定义分类

文章必须从以下分类中选择(参考 `_config.yml`):

- **后端开发** - 服务端、API、数据库、系统设计
- **前端开发** - JavaScript、Vue、React、CSS
- **嵌入式** - STM32、ROS、硬件编程
- **数据工程** - 大数据、机器学习、数据分析
- **工业控制** - C#/.NET、上位机、DCS
- **教程** - 系列教程、入门指南

#### 标签建议

使用具体技术名称:
- 语言: `C++`, `Python`, `JavaScript`, `C#`, `Rust`
- 框架: `Vue`, `.NET Core`, `Django`, `React`
- 工具: `Docker`, `Kubernetes`, `Git`
- 概念: `函数式编程`, `微服务`, `设计模式`

### 书籍格式（`_books/<slug>.md`）

```yaml
---
title: 书籍标题
slug: unique-identifier
author: 作者名
description: 简介
order: 1
---

书籍介绍...
```

### 章节格式（`_chapters/<book-slug>-NN.md`）

```yaml
---
title: 第N章：标题
book: unique-identifier
order: 1
summary: 章节摘要
permalink: /books/<slug>/chapter/
---

章节内容...
```

### 命名规范

- **文件名**：小写 + 连字符（`kebab-case`）
  - ✓ `2025-01-18-rust-ownership.md`
  - ✗ `2025-01-18-Rust_Ownership.md`

- **文件位置**: 按年份组织
  - ✓ `_posts/2025/2025-01-18-article.md`
  - ✗ `_posts/2025-01-18-article.md` (不在年份目录)

- **Front matter 字段**：下划线分隔（`snake_case`）
  - ✓ `reading_time`, `cover_alt`
  - ✗ `readingTime`, `coverAlt`

### 内容规范

- 标题从 `##` 开始
- 代码块必须指定语言: ` ```python `, ` ```c++ `
- 图片使用绝对路径：`/assets/images/posts/YYYY-MM-DD-slug/image.png`
- 图片文件命名规范见 `assets/images/README.md`
- 内部链接：`/books/cpp-functional-programming/` 或 `/2025/01/18/article-slug/`

### 作者引用规范

**必须使用 `_data/authors.yml` 中定义的 key**:

```yaml
# ✓ 正确 - 使用 authors.yml 中的 key
author: phli

# ✗ 错误 - 不要使用显示名称
author: Phli
author: 老高
```

当前可用的作者 key:
- `zhuxi` (老王)
- `phli` (老高)
- `allen` (没看见)
- `leon` (鱼晓亮)
- `lele` (Lele she)
- `bg` (牛志)

新成员加入需先在 `_data/authors.yml` 添加信息。

---

## 本地预览

```bash
# 安装依赖（首次）
bundle install

# 启动开发服务器
bundle exec jekyll serve --livereload

# 访问 http://localhost:4000

# 构建验证
bundle exec jekyll build --trace
bundle exec jekyll doctor
```

**注意**：修改 `_config.yml` 后需重启服务器。

---

## 常见问题

### 如何同步上游更新？

```bash
git checkout main
git pull upstream main
git push origin main
```

### 提交后发现错误？

**PR 未合并**：
```bash
# 修改文件
git add .
git commit -m "[fix] 修正错误"
git push origin add-my-article  # PR 自动更新
```

**PR 已合并**：
```bash
git checkout -b fix-typo
# 修改文件
git commit -m "[fix] 修正拼写错误"
git push origin fix-typo
# 创建新 PR
```

### Liquid 语法冲突？

C++ 双花括号初始化会与 Liquid 冲突，需用 `{% raw %}` 包裹：

~~~markdown
{% raw %}
\`\`\`cpp
std::vector<int> v = {{1, 2, 3}};
\`\`\`
{% endraw %}
~~~

### 如何引用其他文章？

```markdown
参考：[文章标题](/2025/01/18/article-slug/)
或：[书籍名](/books/book-slug/)
```

---

## 获取帮助

- **文档**：[CLAUDE.md](./CLAUDE.md)
- **仓库**：[https://github.com/wzxzhuxi/wzxzhuxi.github.io](https://github.com/wzxzhuxi/wzxzhuxi.github.io)
- **Issues**：[提交问题](https://github.com/wzxzhuxi/wzxzhuxi.github.io/issues)
- **联系**：[@wzxzhuxi](https://github.com/wzxzhuxi)

---

## 图片资源规范

详细规范见 [`assets/images/README.md`](./assets/images/README.md)

### 目录结构

```
assets/images/
├── authors/              # 作者头像 (如 zhuxi.jpg)
├── posts/
│   └── YYYY-MM-DD-slug/  # 每篇文章独立目录
│       ├── cover.png
│       ├── diagram-*.png
│       └── screenshot-*.png
└── books/
    └── book-slug/        # 每本书独立目录
```

### 命名规则

- **封面图**: `cover.png`
- **图表**: `diagram-{描述}.png` (如 `diagram-database-flow.png`)
- **截图**: `screenshot-{描述}.png` (如 `screenshot-result.png`)
- **禁止**: `1.png`, `image.png`, `图片1.png`

### 格式要求

- 单张图片 < 500KB (使用 TinyPNG 压缩)
- 照片用 JPEG,图表/截图用 PNG
- 内容图宽度 ≤ 1200px

---

## 检查清单

提交前请确认：

### 格式规范
- [ ] 遵循 front matter 规范（snake_case）
- [ ] `author` 使用 `_data/authors.yml` 中的 key
- [ ] `categories` 从预定义分类选择
- [ ] 标题从 `##` 开始
- [ ] 代码块指定语言

### 文件组织
- [ ] 文章放入 `_posts/YYYY/` 目录
- [ ] 图片放入 `assets/images/posts/YYYY-MM-DD-slug/` 目录
- [ ] 图片文件名使用英文描述性命名
- [ ] 图片已压缩(< 500KB)

### 测试验证
- [ ] 本地运行 `bundle exec jekyll serve` 预览正常
- [ ] 运行 `bundle exec jekyll build --trace` 无错误
- [ ] 运行 `bundle exec jekyll doctor` 通过检查
- [ ] 图片路径正确且显示正常

### Git 规范
- [ ] Commit 信息符合格式: `[类型] 简短描述`
- [ ] 无 Liquid 语法冲突

---

感谢你的贡献！🎉
