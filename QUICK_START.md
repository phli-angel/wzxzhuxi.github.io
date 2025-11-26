# 快速开始指南

本文档提供 A_Bit_S Blog 的快速上手指南,包括添加成员、发布文章、撰写书籍和记录学习日志的完整流程。

## 目录

- [环境配置](#环境配置)
- [添加团队成员](#添加团队成员)
- [发布文章](#发布文章)
- [撰写书籍](#撰写书籍)
- [记录学习日志](#记录学习日志)
- [本地预览](#本地预览)
- [提交发布](#提交发布)

---

## 环境配置

### 首次使用

```bash
# 1. 克隆仓库
git clone https://github.com/wzxzhuxi/wzxzhuxi.github.io.git
cd wzxzhuxi.github.io

# 2. 安装依赖
bundle install

# 3. 启动开发服务器
bundle exec jekyll serve --livereload

# 4. 访问预览
# http://localhost:4000
```

### 常用命令

```bash
# 启动开发服务器(自动刷新)
bundle exec jekyll serve --livereload

# 构建网站
bundle exec jekyll build --trace

# 健康检查
bundle exec jekyll doctor
```

**重要提示**: 修改 `_config.yml` 后需重启开发服务器。

---

## 添加团队成员

### 步骤 1: 编辑作者数据

编辑 `_data/authors.yml`:

```yaml
newmember:
  name: NewMember              # 英文名(页面显示)
  display_name: 昵称           # 中文昵称(团队页和日记本显示)
  role: 职位描述
  avatar: /assets/images/authors/newmember.jpg
  bio: 个人简介(1-2 句话)
  areas:
    - 技能领域1
    - 技能领域2
  focus: 近期关注的技术方向
  github: github用户名         # 可选
  non_tech: false              # 是否为非技术成员(true 则不显示日记本)
```

### 步骤 2: 添加头像

将头像文件放到 `assets/images/authors/` 目录:
- 文件名: `newmember.jpg` (与 authors.yml 中的 key 一致)
- 推荐尺寸: 200x200 或更大的正方形
- 格式: JPG 或 PNG

### 步骤 3: 创建日记本页面(仅 IT 成员)

如果是技术成员,创建日记本页面 `diaries/newmember.md`:

```yaml
---
layout: author-diaries
title: 昵称的日记本
permalink: /diaries/newmember/
author_key: newmember
---
```

---

## 发布文章

### 快速发布

```bash
# 1. 复制模板
cp _templates/POST_TEMPLATE.md _posts/2025/2025-11-24-my-article.md

# 2. 编辑文章
vim _posts/2025/2025-11-24-my-article.md
```

### Front Matter 配置

```yaml
---
layout: post
title: "文章标题"
author: zhuxi                  # 使用 authors.yml 中的 key
date: 2025-11-24
categories: [后端开发]         # 从预定义分类选择
tags: [Python, Django, API]
excerpt: >
  1-2 句话的文章摘要,用于列表页显示
cover: /assets/images/posts/2025-11-24-my-article/cover.png  # 可选
---
```

### 预定义分类

请从以下分类中选择(保持一致性):
- **后端开发**: 服务端、API、数据库
- **前端开发**: UI、JavaScript、框架
- **嵌入式**: 单片机、硬件控制
- **数据工程**: 大数据、机器学习
- **工业控制**: 上位机、DCS
- **教程**: 系统性学习指南

### 文章图片管理

```bash
# 创建文章专属图片目录
mkdir -p assets/images/posts/2025-11-24-my-article

# 添加图片
cp cover.png assets/images/posts/2025-11-24-my-article/
cp diagram-architecture.png assets/images/posts/2025-11-24-my-article/
```

**命名规范**:
- 封面: `cover.png`
- 图表: `diagram-{描述}.png`
- 截图: `screenshot-{描述}.png`
- **禁止**: `1.png`, `图片1.png`, `image.png`

### Markdown 规范

```markdown
---
layout: post
title: "文章标题"
# ... front matter ...
---

## 引言

正文从 h2 开始(h1 由布局自动生成)。

## 主要内容

### 子章节

使用 h3 作为子标题。

## 代码示例

​```python
def hello():
    print("Hello, World!")
​```

## 图片引用

![图片描述](/assets/images/posts/2025-11-24-my-article/diagram-flow.png)

## 结论

总结内容。
```

**重要提示**:
- 正文从 `##` (h2) 开始,不使用 `#` (h1)
- 代码块必须标注语言
- 图片使用绝对路径
- Front matter 使用 `snake_case`

---

## 撰写书籍

### 步骤 1: 创建书籍元数据

在 `_books/` 目录创建书籍文件 `_books/my-book.md`:

```yaml
---
title: 书籍标题
slug: my-book                  # 书籍唯一标识符
author: zhuxi                  # 使用 authors.yml 中的 key
description: 书籍简介(1-2 句话)
cover: /assets/images/books/my-book/cover.png  # 可选
order: 1                       # 在书籍列表中的排序
---

这里可以写书籍的详细介绍、阅读指南等内容。
```

### 步骤 2: 创建章节目录

```bash
mkdir -p _chapters/my-book
```

### 步骤 3: 添加章节

创建章节文件 `_chapters/my-book/01-intro.md`:

```yaml
---
title: 第一章: 章节标题
book: my-book                  # 必须与书籍 slug 一致
order: 1                       # 章节顺序
summary: 本章简介
permalink: /books/my-book/intro/  # 可选,自定义 URL
---

## 章节内容

章节正文从 h2 开始...
```

### 章节命名规范

推荐使用数字前缀方便排序:
- `01-intro.md`
- `02-basics.md`
- `03-advanced.md`

### 书籍图片管理

```bash
# 创建书籍专属图片目录
mkdir -p assets/images/books/my-book

# 添加封面和章节图片
cp cover.png assets/images/books/my-book/
```

---

## 记录学习日志

### 步骤 1: 确认日记本存在

检查 `diaries/` 目录是否有你的日记本页面:
- 如果没有,创建 `diaries/yourname.md` (参考 [添加团队成员](#添加团队成员))

### 步骤 2: 创建日志目录

```bash
mkdir -p _diaries/yourname
```

### 步骤 3: 撰写日志

创建日志文件 `_diaries/yourname/2025-11-24-learning-react.md`:

```yaml
---
layout: post
title: "学习 React Hooks"
author: yourname               # 使用 authors.yml 中的 key
date: 2025-11-24
tags: [学习, React, 前端]
excerpt: 今天深入学习了 React Hooks 的工作原理
---

## 学习内容

今天主要学习了 `useState` 和 `useEffect` 的使用...

## 遇到的问题

在使用 `useEffect` 时遇到了依赖项警告...

## 解决方案

通过仔细阅读文档,发现...

## 总结

今天的收获是...
```

### 日志与文章的区别

| 特性 | 学习日志 | 技术文章 |
|------|---------|---------|
| **目录** | `_diaries/author/` | `_posts/year/` |
| **风格** | 个人化、过程记录 | 正式、系统性 |
| **长度** | 灵活,可短可长 | 通常较完整 |
| **目的** | 记录学习过程 | 分享技术方案 |
| **分类** | 按作者分类 | 按技术领域分类 |

---

## 本地预览

### 启动开发服务器

```bash
bundle exec jekyll serve --livereload
```

访问 `http://localhost:4000` 查看:
- 首页: `/`
- 文章列表: `/`
- 书籍列表: `/books/`
- 日记本: `/diaries/`
- 团队页面: `/team/`

### 验证构建

```bash
# 完整构建(检查错误)
bundle exec jekyll build --trace

# 健康检查(检查断链、缺失资源)
bundle exec jekyll doctor
```

**发布前必须**:
1. `jekyll build --trace` 零警告
2. `jekyll doctor` 通过
3. 浏览器检查链接和样式

---

## 提交发布

### Git 提交规范

```bash
# 查看状态
git status

# 添加文件
git add _posts/2025/2025-11-24-my-article.md
git add assets/images/posts/2025-11-24-my-article/

# 提交(使用规范格式)
git commit -m "[feat] 添加文章: 我的技术文章标题"

# 推送到远程
git push origin main
```

### 提交类型标签

| 标签 | 说明 | 示例 |
|------|------|------|
| `[feat]` | 新增内容 | `[feat] 添加文章: Rust 所有权详解` |
| `[fix]` | 修复错误 | `[fix] 修正图片链接错误` |
| `[docs]` | 文档更新 | `[docs] 更新 README` |
| `[style]` | 样式调整 | `[style] 优化移动端布局` |
| `[refactor]` | 重构 | `[refactor] 重构书籍系统` |
| `[chore]` | 杂项 | `[chore] 更新依赖` |

### 多文件提交示例

```bash
# 添加新书籍的所有文件
git add _books/my-book.md
git add _chapters/my-book/
git add assets/images/books/my-book/
git commit -m "[feat] 添加书籍: 我的新书"
git push origin main
```

### 部署流程

1. **推送到 main 分支**
   ```bash
   git push origin main
   ```

2. **GitHub Actions 自动构建**
   - 等待 1-2 分钟
   - 查看 Actions 页面确认构建成功

3. **访问网站验证**
   - 访问 `https://wzxzhuxi.github.io/`
   - 检查新内容是否正确显示

---

## 常见问题

### Q: 修改 `_config.yml` 后没有生效?
A: 必须重启开发服务器: `Ctrl+C` 停止,然后重新运行 `bundle exec jekyll serve --livereload`

### Q: 图片显示不出来?
A: 检查:
1. 路径是否正确(必须以 `/assets/` 开头)
2. 文件是否存在
3. 文件名大小写是否匹配

### Q: 书籍的章节列表为空?
A: 检查:
1. 章节文件的 `book:` 字段是否与书籍的 `slug:` 字段完全一致
2. 章节文件是否有 `order:` 字段
3. 章节文件是否在 `_chapters/` 目录下

### Q: 日记本不显示?
A: 检查:
1. `_data/authors.yml` 中该成员是否设置了 `non_tech: true`
2. `diaries/` 目录下是否有对应的页面文件
3. 页面文件的 `author_key:` 是否正确

### Q: 提交后网站没有更新?
A:
1. 查看 GitHub Actions 构建状态
2. 等待 3-5 分钟后再刷新
3. 清除浏览器缓存

---

## 快速参考

### 文件路径速查

```
添加成员:      _data/authors.yml
发布文章:      _posts/YYYY/YYYY-MM-DD-title.md
创建书籍:      _books/book-slug.md
添加章节:      _chapters/book-slug/NN-title.md
撰写日志:      _diaries/author-key/YYYY-MM-DD-title.md
日记本页面:    diaries/author-key.md
文章图片:      assets/images/posts/YYYY-MM-DD-title/
书籍图片:      assets/images/books/book-slug/
日志图片:      assets/images/diaries/author-key/
作者头像:      assets/images/authors/author-key.jpg
```

### 模板文件

所有模板位于 `_templates/` 目录:
- `POST_TEMPLATE.md` - 文章模板
- `BOOK_TEMPLATE.md` - 书籍模板
- `CHAPTER_TEMPLATE.md` - 章节模板
- `DIARY_TEMPLATE.md` - 日志模板

### 有用的命令

```bash
# 查找文章
ls _posts/2025/

# 查找书籍
ls _books/

# 查找某本书的章节
ls _chapters/cpp-functional-programming/

# 查找某作者的日志
ls _diaries/zhuxi/

# 搜索关键词
grep -r "React" _posts/
grep -r "函数式" _books/

# 统计内容
find _posts/ -name "*.md" | wc -l  # 文章数
find _diaries/ -name "*.md" | wc -l  # 日志数
```

---

## 更多资源

- **项目架构**: 查看 `CLAUDE.md` 了解详细的技术架构
- **协作规范**: 查看 `CONTRIBUTING.md` 了解 PR 流程
- **主 README**: 查看 `README.md` 了解完整项目信息
- **图片规范**: 查看 `assets/images/README.md`

---

**Happy Writing! 🚀**
