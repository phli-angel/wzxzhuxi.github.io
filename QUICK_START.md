# 快速开始指南

## 🆕 添加新成员

### 1. 编辑作者数据文件

编辑 `_data/authors.yml`,添加新成员信息:

```bash
nano _data/authors.yml
```

添加内容(使用小写英文 key):

```yaml
newmember:                    # 唯一标识符,小写英文
  name: NewMember             # 英文名(显示在页面上)
  display_name: 昵称          # 中文昵称(仅用于 team.md)
  role: 职位描述
  avatar: /assets/images/authors/newmember.jpg
  bio: 个人简介
  areas:
    - 技能1
    - 技能2
    - 技能3
  focus: 近期关注的技术
  github: github用户名       # 可选
```

### 2. 上传作者头像(可选)

```bash
# 将头像放到 assets/images/authors/ 目录
cp /path/to/photo.jpg assets/images/authors/newmember.jpg
```

建议尺寸: 200x200px 正方形

### 3. 更新 team.md(可选)

如果要在团队页面展示,编辑 `team.md` 添加成员介绍。

---

## 📝 添加新文章

### 1. 复制文章模板

```bash
# 复制模板
cp _posts/POST_TEMPLATE.md _posts/2025/2025-11-20-my-article.md

# 或直接创建
nano _posts/2025/2025-11-20-my-article.md
```

### 2. 填写 Front Matter

```yaml
---
layout: post
title: "文章标题"
author: phli                  # 使用 _data/authors.yml 中的 key
date: 2025-11-20
categories: [工业控制]        # 从预定义分类选择
tags: [C#, .NET Core, Database]
excerpt: 这是一篇关于...的文章
cover: /assets/images/posts/2025-11-20-my-article/cover.png  # 可选
---
```

### 3. 预定义分类(必须从中选择)

- **后端开发** - 服务端、API、数据库
- **前端开发** - JavaScript、Vue、React
- **嵌入式** - STM32、ROS、硬件
- **数据工程** - 大数据、机器学习
- **工业控制** - C#/.NET、上位机、DCS
- **教程** - 系列教程、入门指南

### 4. 编写内容

```markdown
## 第一个标题

正文内容...

```python
# 代码示例(必须标注语言)
def hello():
    print("Hello")
```

![图片描述](/assets/images/posts/2025-11-20-my-article/diagram.png)
*图片说明*
```

### 5. 添加图片(如需要)

```bash
# 创建文章图片目录
mkdir -p assets/images/posts/2025-11-20-my-article

# 复制图片
cp /path/to/image.png assets/images/posts/2025-11-20-my-article/diagram-flow.png
```

**图片命名规则**:
- 封面: `cover.png`
- 图表: `diagram-{描述}.png`
- 截图: `screenshot-{描述}.png`
- **禁止**: `1.png`, `image.png`

### 6. 本地预览

```bash
bundle exec jekyll serve --livereload
# 访问 http://localhost:4000
```

### 7. 提交

```bash
git add _posts/2025/2025-11-20-my-article.md
git add assets/images/posts/2025-11-20-my-article/  # 如有图片
git commit -m "[feat] 添加文章: 文章标题"
git push origin main
```

---

## 📚 添加新书籍

### 方式一: 单本新书

#### 1. 创建书籍元数据

```bash
nano _books/my-new-book.md
```

内容:

```yaml
---
title: 书籍标题
slug: my-new-book           # URL 使用的标识符
description: 书籍简介描述
cover: /assets/images/books/my-new-book/cover.png  # 可选
order: 3                     # 排序(数字越小越靠前)
author: zhuxi                # 使用 _data/authors.yml 中的 key
github_repo: "https://github.com/wzxzhuxi/repo-name"  # 可选
---

## 关于本书

书籍详细介绍...

## 学习路径

### 基础篇
- 第一章: ...
- 第二章: ...
```

#### 2. 创建章节文件

```bash
nano _chapters/my-new-book-01.md
```

内容:

```yaml
---
title: 第一章: 章节标题
book: my-new-book           # 必须匹配书籍的 slug
order: 1                    # 章节顺序
summary: 章节摘要
permalink: /books/my-new-book/chapter-01/  # 可选,自定义 URL
---

## 章节内容

正文...
```

#### 3. 添加更多章节

```bash
# 第二章
nano _chapters/my-new-book-02.md

# 第三章
nano _chapters/my-new-book-03.md

# ...依此类推
```

#### 4. 添加书籍封面(可选)

```bash
mkdir -p assets/images/books/my-new-book
cp /path/to/cover.png assets/images/books/my-new-book/cover.png
```

### 方式二: 批量创建章节

使用脚本快速创建多个章节:

```bash
# 创建 10 个章节模板
for i in {01..10}; do
cat > _chapters/my-new-book-$i.md <<EOF
---
title: 第${i}章: 标题待定
book: my-new-book
order: $i
summary: 章节摘要待定
---

## 章节内容

待编写...
EOF
done
```

### 5. 本地预览

```bash
bundle exec jekyll serve
# 访问 http://localhost:4000/books/
# 点击你的书籍查看章节列表
```

### 6. 提交

```bash
git add _books/my-new-book.md
git add _chapters/my-new-book-*.md
git add assets/images/books/my-new-book/  # 如有图片
git commit -m "[feat] 添加书籍: 书籍标题"
git push origin main
```

---

## ✅ 提交前检查清单

### 文章检查
- [ ] 文件放在 `_posts/YYYY/` 目录
- [ ] 文件名格式: `YYYY-MM-DD-slug.md`
- [ ] `author` 使用 `_data/authors.yml` 中的 key
- [ ] `categories` 从预定义分类选择
- [ ] 代码块标注了语言
- [ ] 图片路径正确
- [ ] 本地预览正常

### 书籍检查
- [ ] 书籍文件在 `_books/` 目录
- [ ] 章节文件在 `_chapters/` 目录
- [ ] 所有章节的 `book` 字段匹配书籍 `slug`
- [ ] 章节 `order` 字段连续
- [ ] 本地预览章节列表正确

### 构建测试
```bash
bundle exec jekyll build --trace  # 必须无错误
bundle exec jekyll doctor         # 必须通过检查
```

---

## 🔧 常用命令

```bash
# 启动开发服务器(自动刷新)
bundle exec jekyll serve --livereload

# 构建网站
bundle exec jekyll build

# 健康检查
bundle exec jekyll doctor

# 查看所有文章
ls _posts/2025/

# 查看所有书籍
ls _books/

# 查看所有章节
ls _chapters/

# 搜索内容
grep -r "关键词" _posts/
grep -r "关键词" _chapters/
```

---

## 📖 完整文档

- **协作流程**: `CONTRIBUTING.md`
- **图片规范**: `assets/images/README.md`
- **项目架构**: `CLAUDE.md`
- **改进记录**: `ARCHITECTURE_IMPROVEMENTS.md`
- **文章模板**: `_posts/POST_TEMPLATE.md`

---

## 🆘 常见问题

**Q: 如何修改已发布的文章?**
```bash
# 直接编辑文件
nano _posts/2025/2025-11-20-article.md

# 提交修改
git commit -am "[fix] 修正文章错误"
git push
```

**Q: 如何删除文章?**
```bash
git rm _posts/2025/2025-11-20-article.md
git commit -m "[chore] 删除文章"
git push
```

**Q: 如何添加章节到已有书籍?**
```bash
# 1. 找到书籍的 slug
grep "slug:" _books/*.md

# 2. 创建新章节(使用该 slug)
nano _chapters/book-slug-05.md

# 3. 设置正确的 order 序号
```

**Q: 图片不显示怎么办?**
- 检查路径是否以 `/` 开头: `/assets/images/...`
- 检查文件名是否匹配(区分大小写)
- 检查文件是否在正确目录

**Q: 构建失败怎么办?**
```bash
# 查看详细错误信息
bundle exec jekyll build --trace

# 常见错误:
# - Liquid 语法错误: 检查 {{ }} 和 {% %}
# - YAML 错误: 检查 front matter 格式
# - 文件路径错误: 检查文件是否存在
```

---

## 🚀 快速示例

### 5 分钟发布一篇文章

```bash
# 1. 创建文章
cat > _posts/2025/2025-11-20-hello-world.md <<EOF
---
layout: post
title: "Hello World"
author: phli
date: 2025-11-20
categories: [教程]
tags: [入门]
excerpt: 我的第一篇文章
---

## 开始

这是我的第一篇文章!
EOF

# 2. 本地预览
bundle exec jekyll serve

# 3. 提交
git add _posts/2025/2025-11-20-hello-world.md
git commit -m "[feat] 添加文章: Hello World"
git push
```

完成! 🎉
