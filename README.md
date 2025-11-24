# A_Bit_S Blog

> 🌐 切换语言 / Language: [English](README_EN.md)

## 项目简介

A_Bit_S Blog 是团队的技术内容发布平台，基于 Jekyll 构建并部署在 GitHub Pages (`https://wzxzhuxi.github.io/`)。

支持文章、书籍(多章节长文)两种内容形式，采用统一的作者数据管理系统，实现多人协作发布。

## 快速开始

### 本地开发

```bash
# 1. 安装依赖
bundle install

# 2. 启动开发服务器(带自动刷新)
bundle exec jekyll serve --livereload

# 3. 访问预览
# http://localhost:4000
```

### 构建验证

```bash
# 构建网站
bundle exec jekyll build --trace

# 健康检查
bundle exec jekyll doctor
```

**注意**: 修改 `_config.yml` 后需重启开发服务器。

## 项目结构

```
.
├── _data/
│   └── authors.yml           # 作者数据库(统一管理所有成员信息)
├── _posts/
│   └── YYYY/                 # 按年份组织的文章
│       └── YYYY-MM-DD-slug.md
├── _templates/               # 模板目录(不会被Jekyll处理)
│   ├── POST_TEMPLATE.md
│   ├── BOOK_TEMPLATE.md
│   ├── CHAPTER_TEMPLATE.md
│   └── DIARY_TEMPLATE.md
├── _books/                   # 书籍元数据文件(扁平结构)
│   ├── cpp-functional-programming.md
│   └── book-slug.md
├── _chapters/                # 章节按书籍分子目录
│   ├── cpp-functional-programming/
│   │   ├── 01-setup.md
│   │   ├── 02-lambda.md
│   │   └── ...
│   └── book-slug/
│       └── 01-intro.md
├── _diaries/                 # 学习日志按作者分子目录
│   ├── zhuxi/
│   │   ├── 2025-11-21-first-diary.md
│   │   └── 2025-11-22-second-diary.md
│   └── author-key/
│       └── YYYY-MM-DD-title.md
├── diaries/                  # 日记本页面
│   ├── zhuxi.md
│   ├── phli.md
│   └── author-key.md
├── _layouts/                 # 页面布局模板
├── assets/
│   ├── css/style.scss        # 全局样式
│   └── images/               # 图片资源
│       ├── authors/          # 作者头像
│       ├── posts/YYYY-MM-DD-slug/  # 文章图片(按文章分目录)
│       └── books/book-slug/  # 书籍图片
├── books.md                  # 书籍列表页
├── diaries.md                # 学习日志主页(所有成员的日记本)
├── _config.yml               # 站点配置
├── QUICK_START.md            # 快速操作指南
├── CONTRIBUTING.md           # 协作规范
└── CLAUDE.md                 # 项目架构文档
```

## 内容管理

### 添加新成员

编辑 `_data/authors.yml`:

```yaml
newmember:
  name: NewMember              # 英文名(页面显示)
  display_name: 昵称           # 中文昵称(团队页显示)
  role: 职位描述
  avatar: /assets/images/authors/newmember.jpg
  bio: 个人简介
  areas:
    - 技能1
    - 技能2
  focus: 近期关注的技术
  github: github用户名         # 可选
  non_tech: false              # 是否为非技术成员(true 则不显示日记本)
```

### 添加新文章

1. 创建文件: `_posts/2025/2025-MM-DD-title.md`
2. 填写 front matter:

```yaml
---
layout: post
title: "文章标题"
author: phli                   # 使用 authors.yml 中的 key
date: 2025-11-20
categories: [工业控制]         # 从预定义分类选择
tags: [C#, .NET Core, Database]
excerpt: 1-2 句话的文章摘要
cover: /assets/images/posts/2025-11-20-title/cover.png  # 可选
---
```

**预定义分类**:
- 后端开发
- 前端开发
- 嵌入式
- 数据工程
- 工业控制
- 教程

### 添加学习日志

1. 创建日记本页面(如果还没有): `diaries/author-key.md`

```yaml
---
layout: author-diaries
title: [昵称]的日记本
permalink: /diaries/[author-key]/
author_key: author-key         # 必须与 authors.yml 中的 key 一致
---
```

2. 创建日志文件: `_diaries/author-key/YYYY-MM-DD-title.md`

```yaml
---
layout: post
title: "日志标题"
author: author-key             # 使用 authors.yml 中的 key
date: 2025-11-24
tags: [学习, 技术探索]
excerpt: 简短摘要
---
```

**注意**:
- 只有 IT 技术成员才会显示日记本(非技术成员设置 `non_tech: true`)
- 日志文件按作者分目录存放: `_diaries/author-key/`
- 日志采用与文章相同的 Markdown 格式

### 添加新书籍

1. 复制模板创建书籍目录:

```bash
cp -r _templates/BOOK_TEMPLATE _books/my-book
cd _books/my-book
mv book.md my-book.md
```

2. 编辑书籍主文件: `_books/my-book/my-book.md`

```yaml
---
title: 书籍标题
slug: my-book                  # 必须与目录名和文件名一致
author: zhuxi                  # 使用 authors.yml 中的 key
description: 书籍简介
cover: /assets/images/books/my-book/cover.png  # 可选
order: 1                       # 排序
---
```

3. 在同一目录创建章节: `_books/my-book/my-book-01-intro.md`

```yaml
---
title: 第一章: 章节标题
book: my-book                  # 必须匹配书籍 slug
order: 1                       # 章节顺序
summary: 章节摘要
---
```

## 图片管理

### 目录结构

```
assets/images/
├── authors/           # 作者头像(如 zhuxi.jpg)
├── posts/
│   └── YYYY-MM-DD-slug/  # 每篇文章独立目录
│       ├── cover.png
│       ├── diagram-flow.png
│       └── screenshot-result.png
├── books/
│   └── book-slug/     # 每本书独立目录
└── diaries/
    └── author-key/    # 每位作者的日志图片
```

### 命名规范

- 封面: `cover.png`
- 图表: `diagram-{描述}.png`
- 截图: `screenshot-{描述}.png`
- **禁止**: `1.png`, `image.png`, `图片1.png`

详见 `assets/images/README.md`

## Markdown 规范

- 正文从 `##` (h2) 开始,不使用 `#` (h1)
- 代码块必须标注语言: ` ```python `, ` ```c++ `
- 图片使用绝对路径: `/assets/images/posts/...`
- Front matter 使用 `snake_case`: `reading_time`, `cover_alt`

## Git 提交规范

```bash
# 格式
[类型] 简短描述

# 类型标签
[feat]     # 新增内容(文章、书籍、功能)
[fix]      # 修复错误(链接、格式、拼写)
[docs]     # 文档更新
[style]    # 样式调整
[refactor] # 重构
[chore]    # 杂项

# 示例
git commit -m "[feat] 添加文章: Rust 所有权详解"
git commit -m "[fix] 修正图片链接错误"
```

## 部署

项目通过 GitHub Pages 自动部署:

1. 推送到 `main` 分支
2. GitHub Actions 自动构建
3. 发布到 `https://wzxzhuxi.github.io/`

**注意**:
- `_site/` 目录是构建输出,已被 `.gitignore` 排除
- 不要手动编辑或提交 `_site/` 目录

## 核心文档

| 文档 | 说明 |
|------|------|
| `QUICK_START.md` | 快速操作指南(添加成员/文章/书籍) |
| `CONTRIBUTING.md` | 协作规范(PR 流程、检查清单) |
| `CLAUDE.md` | 项目架构和设计原则 |
| `ARCHITECTURE_IMPROVEMENTS.md` | 架构改进记录 |
| `assets/images/README.md` | 图片规范 |
| `_templates/` | 所有模板文件(文章、书籍、章节、日志) |

## 常用命令

```bash
# 开发
bundle exec jekyll serve --livereload

# 构建
bundle exec jekyll build --trace

# 检查
bundle exec jekyll doctor

# 查看内容
ls _posts/2025/                    # 文章列表
ls _books/                         # 书籍列表
ls _chapters/cpp-functional-programming/  # 查看某本书的章节
ls _diaries/zhuxi/                 # 查看作者的学习日志
ls diaries/                        # 日记本页面列表

# 搜索
grep -r "关键词" _posts/
grep -r "关键词" _books/
```

## 技术栈

- **静态站点生成**: Jekyll 3.10+
- **主题**: Minima (自定义样式)
- **代码高亮**: Prism.js (Tomorrow Night 主题)
- **托管**: GitHub Pages
- **依赖管理**: Bundler

## 贡献

参考 `CONTRIBUTING.md` 了解:
- Fork/PR 工作流
- 文章提交模板
- 图片规范
- 本地测试要求
- 代码审核标准

## 团队

访问 [/team/](/team/) 页面了解团队成员。

## License

内容版权归 A_Bit_S 团队所有。
