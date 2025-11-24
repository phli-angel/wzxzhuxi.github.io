---
name: 📝 提交文章
about: 团队成员提交新文章或教程
title: '[文章] '
labels: article
assignees: ''
---

## 📌 文章信息

**标题**:

**作者**: @你的GitHub用户名

**分类**:
- [ ] 后端开发
- [ ] 前端开发
- [ ] 嵌入式
- [ ] 数据工程
- [ ] 工业控制
- [ ] 教程

**标签**: (用逗号分隔,例如: C++, Docker, Kubernetes)


**预计字数**:

---

## 📄 文章概要

<!-- 用 2-3 句话描述文章主要内容 -->


---

## ✅ 提交前检查清单

### 内容质量
- [ ] 文章已完成,无明显语法错误
- [ ] 代码示例已测试验证
- [ ] 技术细节准确无误
- [ ] 适合目标读者水平

### 格式规范
- [ ] Front matter 包含所有必需字段(title, author, categories, tags, excerpt)
- [ ] 使用 `author: {key}` 引用 `_data/authors.yml` 中的作者
- [ ] 分类从预定义列表选择(见 `_config.yml`)
- [ ] 文章开头从 `##` (h2) 开始,不使用 `#` (h1)
- [ ] 代码块使用三个反引号并标注语言: ` ```python `

### 图片资源
- [ ] 图片已放入 `assets/images/posts/{文章文件名}/` 目录
- [ ] 图片文件名使用英文描述性命名(如 `diagram-flow.png`)
- [ ] 单张图片小于 500KB
- [ ] 文章中图片路径正确

### 本地测试
- [ ] 已运行 `bundle exec jekyll serve --livereload` 本地预览
- [ ] 已运行 `bundle exec jekyll build --trace` 无错误
- [ ] 已运行 `bundle exec jekyll doctor` 通过检查
- [ ] 文章在浏览器中显示正常(包括代码高亮、图片)

### Git 规范
- [ ] 文件已放入 `_posts/YYYY/` 目录
- [ ] 文件名格式: `YYYY-MM-DD-slug.md`
- [ ] Commit message 格式: `[feat] 添加文章: 文章标题`
- [ ] 已创建 Pull Request

---

## 📎 附加信息

**文章文件路径**: `_posts/2025/2025-MM-DD-文章slug.md`

**预览链接**: (PR 创建后填写)

**备注**:


---

## 审核要点(审核人员填写)

- [ ] 内容质量达标
- [ ] 技术准确性
- [ ] 代码示例可运行
- [ ] 格式符合规范
- [ ] 图片资源完整
- [ ] 无拼写/语法错误
