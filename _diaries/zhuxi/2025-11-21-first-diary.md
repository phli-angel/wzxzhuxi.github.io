---
title: "第一篇学习日志"
date: 2025-11-21
author: zhuxi
mood: 💡 充实
weather: ☀️ 晴朗
location: 家中
excerpt: >
  今天为博客添加了学习日志功能,可以记录日常学习过程和技术探索了。
---

## 今天做了什么

今天为 A_Bit_S Blog 添加了全新的学习日志功能!

现在我们的博客有三个主要板块:
- **文章** - 完整的技术文章和教程
- **书籍** - 系列教程和长篇内容
- **日记** - 学习日志和技术笔记

## 技术实现

实现日记功能主要做了以下工作:

1. 在 `_config.yml` 中添加了 `diaries` 集合
2. 创建了 `_diaries` 目录存放日志文件
3. 设计了 `diary.html` 和 `author-diaries.html` 布局模板
4. 创建了两级结构:
   - `/diaries/` - 展示所有作者的日记本
   - `/diaries/{author}/` - 单个作者的日志列表
5. 更新了导航菜单

## 学到的东西

Jekyll 的集合(Collections)功能很强大,可以灵活地组织不同类型的内容。通过 Liquid 的 `where` 过滤器可以轻松实现按作者分组的功能。

关键代码:
```liquid
{% raw %}
{% assign author_diaries = site.diaries | where: "author", author_key | sort: "date" | reverse %}
{% endraw %}
```

## 收获与思考

这种学习日志的形式很适合记录日常的学习过程,比完整的技术文章更轻量,可以快速记录学习心得和遇到的问题。

希望团队成员都能养成记录学习日志的习惯!
