---
layout: post
title: "文章标题"
author: your_author_key  # 必须是 _data/authors.yml 中的 key (如 zhuxi, phli, lele)
date: YYYY-MM-DD
categories: [分类名]  # 从预定义分类选择: 后端开发/前端开发/嵌入式/数据工程/工业控制/教程
tags: [标签1, 标签2, 标签3]  # 技术名称,用于搜索和分组
excerpt: 1-2 句话的文章摘要,显示在列表页和社交分享
cover: /assets/images/posts/YYYY-MM-DD-slug/cover.png  # 可选,封面图
---

## 引言

简短介绍文章背景和要解决的问题。

## 主要内容

### 小节标题

正文内容...

```python
# 代码示例,注意语言标记
def example():
    return "Hello, World!"
```

### 图片示例

![图片描述](/assets/images/posts/YYYY-MM-DD-slug/diagram-example.png)
*可选的图片说明文字*

## 总结

总结文章要点。

---

## 使用说明

1. **复制此模板**: 不要直接修改此文件
2. **重命名**: `YYYY-MM-DD-your-article-slug.md`
3. **放置位置**: `_posts/YYYY/` 目录
4. **填写 Front Matter**: 所有必需字段
5. **创建图片目录**: `assets/images/posts/YYYY-MM-DD-your-article-slug/`
6. **本地预览**: `bundle exec jekyll serve`
7. **提交 PR**: 参考 CONTRIBUTING.md

## Front Matter 字段说明

- `layout`: 固定为 `post`
- `title`: 文章标题(显示在页面顶部)
- `author`: 作者 key,必须在 `_data/authors.yml` 中定义
- `date`: 发布日期,格式 YYYY-MM-DD
- `categories`: 数组,只能从 `_config.yml` 预定义分类选择
- `tags`: 数组,自由标签,建议使用技术名称
- `excerpt`: 摘要,用于列表页和 SEO
- `cover`: 可选,封面图路径

## 格式要点

- 文章正文从 `##` (h2) 开始,不使用 `#` (h1)
- 代码块必须标注语言: ` ```python `, ` ```javascript `, ` ```c++ `
- 图片使用绝对路径: `/assets/images/...`
- 图片后紧跟 `*说明文字*` 会自动格式化为图注

## 预定义分类

- 后端开发
- 前端开发
- 嵌入式
- 数据工程
- 工业控制
- 教程

## 标签建议

使用具体的技术名称作为标签:
- 语言: Python, C++, JavaScript, C#, Rust
- 框架: Vue, React, .NET Core, Django
- 工具: Docker, Kubernetes, Git, VS Code
- 概念: 函数式编程, 微服务, 设计模式
