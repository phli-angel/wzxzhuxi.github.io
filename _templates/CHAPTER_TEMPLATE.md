---
title: 第N章:章节标题
book: your-book-slug        # 必须匹配书籍的 slug 字段
order: 1                    # 章节顺序
summary: 章节摘要,1-2句话概括本章内容
---

## 章节概述

简要介绍本章要讲解的内容和学习目标...

## 核心概念

### 概念1

详细讲解第一个核心概念...

```python
# 示例代码
def example():
    pass
```

### 概念2

详细讲解第二个核心概念...

## 实践示例

通过实际案例演示如何应用本章知识...

## 小结

总结本章的关键要点:
- 要点1
- 要点2
- 要点3

## 练习题

1. 练习题1
2. 练习题2

---

**使用说明**:

1. 确保书籍文件已创建在 `_books/your-book-slug.md`

2. 确保章节目录已创建:
   ```bash
   mkdir -p _chapters/your-book-slug
   ```

3. 复制此模板到章节目录:
   ```bash
   cp _templates/CHAPTER_TEMPLATE.md _chapters/your-book-slug/01-chapter-name.md
   ```

4. 编辑章节文件:
   - `title`: 章节标题
   - `book`: **必须**与书籍的 `slug` 字段完全一致
   - `order`: 章节顺序(用于排序)
   - `summary`: 章节摘要(显示在章节列表)

5. 章节文件命名建议:
   - 简洁格式: `01-introduction.md`, `02-basics.md`, `03-advanced.md`
   - 不需要包含书籍前缀

6. 章节 URL 示例:
   - 文件: `_chapters/cpp-functional-programming/01-introduction.md`
   - URL: `/books/01-introduction/`
