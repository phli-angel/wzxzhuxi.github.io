# 架构改进文档

本文档记录了对 A_Bit_S Blog 多人协作架构的改进。

## 改进时间
2025-11-19

## 改进动机
作为多人博客系统,原有设计存在以下问题:
1. 作者系统混乱(字符串而非数据引用)
2. 缺少分类和标签系统
3. 图片管理无规范
4. 文章目录结构不支持扩展
5. 缺少内容提交模板

## 已完成改进

### 1. 规范化作者系统 ✅

**问题**:
- `author: Phli` 使用简单字符串,拼写错误导致关联失败
- 无法统一管理作者信息(头像、简介、社交链接)

**解决方案**:
- 创建 `_data/authors.yml` 集中管理作者数据
- 文章使用 `author: phli` 引用 key
- 支持扩展字段:头像、简介、领域、GitHub

**文件**:
- `_data/authors.yml` - 作者数据库,包含 6 名团队成员

**示例**:
```yaml
# _data/authors.yml
phli:
  name: Phli
  display_name: 老高
  role: 上位机工程师
  avatar: /assets/images/authors/phli.jpg
  areas: [C#/.NET, 工业控制]
```

```yaml
# 文章 front matter
author: phli  # 引用 key
```

### 2. 分类和标签系统 ✅

**问题**:
- 无法按技术栈筛选文章
- 内容孤岛化

**解决方案**:
- `_config.yml` 定义 6 个预定义分类
- 允许自由标签但建议使用技术名称

**预定义分类**:
- 后端开发
- 前端开发
- 嵌入式
- 数据工程
- 工业控制
- 教程

**文件**:
- `_config.yml:27-38` - 分类和标签配置

### 3. 图片资源规范 ✅

**问题**:
- 多人上传图片命名冲突
- 无组织结构

**解决方案**:
- 按文章/书籍分目录
- 强制描述性命名
- 文件大小限制 < 500KB

**目录结构**:
```
assets/images/
├── authors/              # 作者头像
├── posts/
│   └── YYYY-MM-DD-slug/  # 每篇文章独立目录
│       ├── cover.png
│       ├── diagram-flow.png
│       └── screenshot-result.png
└── books/
    └── book-slug/
```

**命名规则**:
- 封面: `cover.png`
- 图表: `diagram-{描述}.png`
- 截图: `screenshot-{描述}.png`
- **禁止**: `1.png`, `image.png`, `图片1.png`

**文件**:
- `assets/images/README.md` - 完整图片规范文档
- `assets/images/{authors,posts,books}/` - 目录结构

### 4. 按年份组织文章 ✅

**问题**:
- 所有文章堆在 `_posts/` 根目录
- 100+ 文章后难以管理

**解决方案**:
- 文章按年份分目录: `_posts/YYYY/`
- 配置 permalink 保持 URL 不变

**目录结构**:
```
_posts/
├── 2024/
│   └── ...
└── 2025/
    └── 2025-11-15-net-core-database.md
```

**文件**:
- `_config.yml:55` - 添加 permalink 配置
- `_posts/2025/` - 2025 年文章目录

### 5. 内容提交模板 ✅

**问题**:
- 新成员不知道格式要求
- 格式不统一

**解决方案**:
- GitHub Issue 模板
- Pull Request 模板
- 文章模板文件

**文件**:
- `.github/ISSUE_TEMPLATE/article-submission.md` - Issue 提交模板
- `.github/PULL_REQUEST_TEMPLATE.md` - PR 模板
- `_posts/POST_TEMPLATE.md` - 文章模板

### 6. 更新协作指南 ✅

**内容**:
- 补充作者引用规范
- 补充分类/标签说明
- 补充图片规范链接
- 更新检查清单

**文件**:
- `CONTRIBUTING.md` - 完整更新

## 文件清单

### 新增文件
- `_data/authors.yml` - 作者数据库
- `assets/images/README.md` - 图片规范
- `.github/ISSUE_TEMPLATE/article-submission.md` - Issue 模板
- `.github/PULL_REQUEST_TEMPLATE.md` - PR 模板
- `_posts/POST_TEMPLATE.md` - 文章模板
- `ARCHITECTURE_IMPROVEMENTS.md` - 本文档

### 修改文件
- `_config.yml` - 添加分类配置、permalink 配置
- `_posts/2025-11-15-net-core-database.md` - 更新 author 为 `phli`,添加 categories/excerpt
- `CONTRIBUTING.md` - 补充新规范
- `assets/css/style.scss` - 添加图片 CSS 样式

### 重组目录
- `_posts/` → `_posts/2025/` - 文章按年份组织
- `assets/images/` - 重组为 authors/posts/books 三个子目录

## 使用指南

### 新成员加入流程

1. **添加作者信息**
   ```yaml
   # 编辑 _data/authors.yml
   newmember:
     name: NewMember
     display_name: 显示名称
     role: 职位
     avatar: /assets/images/authors/newmember.jpg
     areas: [技能1, 技能2]
   ```

2. **上传头像**
   - 放置在 `assets/images/authors/newmember.jpg`
   - 建议尺寸 200x200px

3. **撰写文章**
   - 复制 `_posts/POST_TEMPLATE.md`
   - 重命名为 `_posts/2025/2025-MM-DD-slug.md`
   - 使用 `author: newmember`

### 发布文章流程

1. **准备内容**
   - 参考 `_posts/POST_TEMPLATE.md`
   - 图片放入 `assets/images/posts/YYYY-MM-DD-slug/`

2. **本地测试**
   ```bash
   bundle exec jekyll serve
   bundle exec jekyll build --trace
   bundle exec jekyll doctor
   ```

3. **提交 PR**
   - Commit: `[feat] 添加文章: 标题`
   - 填写 PR 模板

4. **代码审核**
   - 团队成员审核内容和格式
   - 通过后合并到 main

## 未来改进建议

### 短期(1-2 周)
- [ ] 添加作者头像占位图
- [ ] 更新 `authors/index.md` 使用 `_data/authors.yml`
- [ ] 创建分类索引页(`/categories/`)
- [ ] 创建标签索引页(`/tags/`)

### 中期(1-2 月)
- [ ] 集成评论系统(Giscus/Utterances)
- [ ] 添加站内搜索(Algolia/Lunr.js)
- [ ] RSS 订阅优化
- [ ] 添加阅读时间估算

### 长期(按需)
- [ ] 多语言支持(中英文)
- [ ] 暗黑模式
- [ ] 访问统计(Google Analytics/Umami)
- [ ] 自动化 CI/CD 检查(Markdownlint、图片压缩检查)

## 向后兼容性

所有改进保持向后兼容:
- 旧的 `author: Phli` 格式仍可工作(但不推荐)
- URL 结构保持不变(`/:year/:month/:day/:title/`)
- 现有章节和书籍无需修改

## 技术负债清理

本次改进清理的技术负债:
- ❌ 字符串作者 → ✅ 数据引用
- ❌ 无分类系统 → ✅ 预定义分类
- ❌ 图片混乱 → ✅ 规范化目录和命名
- ❌ 扁平文章目录 → ✅ 按年份组织
- ❌ 无提交指引 → ✅ 完整模板和文档

## 验证

构建测试通过:
```bash
$ bundle exec jekyll build --trace
Configuration file: /home/fuck/jekyll/_config.yml
            Source: /home/fuck/jekyll
       Destination: /home/fuck/jekyll/_site
      Generating...
       Jekyll Feed: Generating feed for posts
                    done in 0.442 seconds.
```

## 贡献者
- 架构改进: Claude (AI Assistant)
- 审核与需求: @wzxzhuxi

---

**文档版本**: 1.0
**最后更新**: 2025-11-19
