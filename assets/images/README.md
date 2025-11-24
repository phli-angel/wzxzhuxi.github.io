# 图片资源目录规范

## 目录结构

```
assets/images/
├── authors/              # 作者头像
│   ├── zhuxi.jpg
│   ├── phli.jpg
│   └── ...
├── posts/                # 博客文章图片
│   ├── 2025-11-15-net-core-database/
│   │   ├── cover.png
│   │   ├── diagram-01.png
│   │   └── screenshot-connection.png
│   └── 2025-11-20-another-post/
│       └── ...
└── books/                # 书籍相关图片
    ├── cpp-fp/
    │   ├── cover.png
    │   ├── ch01-example.png
    │   └── ...
    └── ...
```

## 命名规范

### 1. 作者头像
- 文件名: `{author_key}.jpg` 或 `.png`
- 示例: `zhuxi.jpg`, `phli.jpg`
- 尺寸建议: 200x200px 或更大(正方形)

### 2. 文章图片
- **目录**: 使用文章文件名作为目录名
  - 文章: `_posts/2025-11-15-net-core-database.md`
  - 图片目录: `assets/images/posts/2025-11-15-net-core-database/`

- **文件命名**:
  - 封面图: `cover.png` 或 `cover.jpg`
  - 图表: `diagram-{描述}.png` (如 `diagram-database-flow.png`)
  - 截图: `screenshot-{描述}.png` (如 `screenshot-login.png`)
  - 示例代码: `code-{描述}.png`
  - 通用图片: 使用描述性英文名称,用连字符分隔

- **禁止命名**:
  - ❌ `1.png`, `2.png`, `image.png`
  - ❌ `屏幕截图.png`, `图片1.png`
  - ✅ `diagram-data-flow.png`, `screenshot-result.png`

### 3. 书籍图片
- **目录**: 使用书籍 slug 作为目录名
  - 书籍: `_books/cpp-functional-programming.md` (slug: `cpp-fp`)
  - 图片目录: `assets/images/books/cpp-fp/`

- **文件命名**:
  - 封面: `cover.png`
  - 章节图片: `ch{NN}-{描述}.png` (如 `ch01-lambda-syntax.png`)

## 图片格式要求

1. **格式选择**:
   - 照片/复杂图像: JPEG (`.jpg`)
   - 图表/截图/简单图像: PNG (`.png`)
   - 矢量图: SVG (`.svg`)

2. **文件大小**:
   - 单张图片不超过 500KB
   - 使用在线工具压缩: TinyPNG, Squoosh

3. **尺寸建议**:
   - 文章内容图: 宽度不超过 1200px
   - 封面图: 1200x630px (适合社交媒体分享)
   - 缩略图: 400x300px

## 在文章中引用图片

### 基本语法
```markdown
![图片描述](/assets/images/posts/2025-11-15-net-core-database/diagram-flow.png)
*可选的图片说明文字*
```

### 带封面的文章
```yaml
---
layout: post
title: 文章标题
cover: /assets/images/posts/2025-11-15-example/cover.png
---
```

### 全宽图片
```markdown
![架构图](/assets/images/posts/2025-11-15-example/architecture.png){:.full-width}
```

## 提交前检查清单

- [ ] 图片已放入对应文章/书籍的目录
- [ ] 文件名使用英文描述性命名
- [ ] 图片已压缩(单张 < 500KB)
- [ ] 文章中的图片路径正确
- [ ] 本地预览图片正常显示

## 常见问题

**Q: 我的文章有很多图片,都要单独命名吗?**
A: 是的。描述性命名让 6 个月后的你和其他团队成员都能快速理解图片内容。

**Q: 可以直接引用外部链接的图片吗?**
A: 不推荐。外部图片可能失效,建议下载到本地。

**Q: 图片太大怎么办?**
A: 使用 [TinyPNG](https://tinypng.com/) 或 [Squoosh](https://squoosh.app/) 压缩。
