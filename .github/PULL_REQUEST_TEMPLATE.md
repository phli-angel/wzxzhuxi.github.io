# Pull Request

## 📝 变更类型
- [ ] 新文章
- [ ] 文章更新/修正
- [ ] 新书籍/章节
- [ ] 网站功能改进
- [ ] Bug 修复
- [ ] 文档更新

## 📌 变更说明

### 新增内容(如适用)
**文章标题**:

**作者**: @

**分类**:

**主要内容**:


### 修改内容(如适用)
**修改原因**:


**影响范围**:


---

## ✅ 提交检查清单

### 基本检查
- [ ] 本地运行 `bundle exec jekyll serve` 无错误
- [ ] 运行 `bundle exec jekyll build --trace` 构建成功
- [ ] 运行 `bundle exec jekyll doctor` 通过检查
- [ ] 浏览器预览显示正常

### 内容检查(文章/章节)
- [ ] Front matter 完整(title, author, categories, tags, excerpt)
- [ ] `author` 字段使用 `_data/authors.yml` 中的 key
- [ ] 分类使用预定义分类
- [ ] 代码块正确使用语言标记
- [ ] 图片路径正确且文件存在
- [ ] 图片文件名使用描述性命名
- [ ] 无拼写/语法错误

### Git 规范
- [ ] Commit message 格式正确: `[类型] 简短描述`
- [ ] 文件放入正确目录(`_posts/YYYY/` 或 `_chapters/`)
- [ ] 图片放入 `assets/images/posts/{slug}/` 目录

---

## 🔗 相关链接

**关联 Issue**: #

**本地预览截图**:
<!-- 粘贴文章/页面截图 -->

**部署后 URL**: (合并后填写)

---

## 📋 审核要点(审核人员)

- [ ] 内容质量
- [ ] 技术准确性
- [ ] 格式规范
- [ ] 构建通过
- [ ] 无破坏性变更
