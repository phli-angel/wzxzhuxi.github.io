---
title: 书籍标题
slug: book-slug-identifier
author: 作者名
description: 书籍简介,1-2句话描述本书的核心内容和目标读者
cover: /assets/images/books/book-slug/cover.png  # 可选,书籍封面图片
order: 1  # 在书籍列表中的排序
---

# 书籍介绍

在这里写书籍的详细介绍,包括:
- 本书适合哪些读者
- 读完本书能学到什么
- 本书的特色和亮点

## 学习路径

本书分为以下几个部分:

1. **基础篇** - 介绍基础概念
2. **进阶篇** - 深入核心技术
3. **实战篇** - 实际项目应用

## 前置知识

阅读本书需要具备以下基础:
- 知识点1
- 知识点2
- 知识点3

## 如何使用本书

建议按顺序阅读章节,每章都配有示例代码和练习题。

---

**使用说明**:

1. 将此文件复制到 `_books/` 目录:
   ```bash
   cp _templates/BOOK_TEMPLATE.md _books/my-book.md
   ```

2. 编辑 `_books/my-book.md` 填写书籍元数据:
   - `slug` 字段是关键,用于关联章节
   - `author` 使用 `_data/authors.yml` 中的 key
   - `order` 控制在书籍列表中的排序

3. 创建章节目录(目录名必须与 slug 一致):
   ```bash
   mkdir _chapters/my-book
   ```

4. 在章节目录下创建章节文件(使用 CHAPTER_TEMPLATE.md):
   ```bash
   cp _templates/CHAPTER_TEMPLATE.md _chapters/my-book/01-introduction.md
   ```

5. 文件结构示例:
   ```
   _books/
   └── my-book.md              # 书籍主文件

   _chapters/
   └── my-book/                # 章节目录(名称=slug)
       ├── 01-introduction.md  # 第1章
       ├── 02-basics.md        # 第2章
       └── 03-advanced.md      # 第3章
   ```
