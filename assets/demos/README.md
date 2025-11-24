# 演示文件使用指南

## 目录结构

```
assets/demos/
├── posts/              # 文章配套演示
│   └── YYYY-MM-DD-slug.html
├── books/              # 书籍配套演示
│   └── book-slug/
│       └── chapter-01.html
└── README.md           # 本文件
```

## 为文章添加演示

### 1. 创建独立 HTML 文件

在 `assets/demos/posts/` 目录下创建与文章同名的 HTML 文件：

```bash
# 文件命名规则：与文章文件名一致
# 例如：_posts/2025/2025-11-20-js-event-delegation-tab.md
# 对应：assets/demos/posts/2025-11-20-js-event-delegation-tab.html
```

### 2. HTML 文件要求

- **完整的 HTML 文档**：包含 `<!DOCTYPE html>`, `<html>`, `<head>`, `<body>`
- **使用相对路径**：图片等资源使用相对路径，如 `assets/images/...`
- **自包含**：所有 CSS 和 JavaScript 都写在 HTML 文件内
- **响应式**：考虑移动端显示效果

### 3. 在文章中引用

#### 方案一：自适应高度（推荐）

iframe 根据内容自动调整高度：

```markdown
## 在线演示

<div style="margin: 20px 0;">
    <iframe
        id="demo-iframe-唯一标识"
        src="/assets/demos/posts/YYYY-MM-DD-slug.html"
        style="width: 100%; border: 2px solid #333; border-radius: 4px;"
        frameborder="0"
        scrolling="no"
        onload="this.style.height=(this.contentWindow.document.body.scrollHeight)+'px'">
    </iframe>
</div>

**💡 提示**：演示说明文字。
```

**关键参数**：
- `id`: 唯一标识符，建议使用 `demo-iframe-YYYY-MM-DD`
- `scrolling="no"`: 禁用滚动条
- `onload`: 加载后自动调整高度为内容高度

#### 方案二：固定高度

适用于高度固定的演示：

```markdown
<iframe
    src="/assets/demos/posts/YYYY-MM-DD-slug.html"
    style="width: 100%; height: 600px; border: 2px solid #333;"
    frameborder="0">
</iframe>
```

### 4. iframe 参数说明

- `src`: 演示文件路径（以 `/` 开头的绝对路径）
- `width`: 建议使用 `100%` 适应容器宽度
- `height`: 根据演示内容调整，常用值：400px-800px
- `frameborder="0"`: 移除边框
- `loading="lazy"`: 延迟加载，优化性能

## 优势

### ✅ 完全隔离
- CSS 不会被文章样式覆盖
- JavaScript 不会与页面其他脚本冲突
- 完全原生的运行环境

### ✅ 易于维护
- 演示代码独立管理
- 可以单独测试和调试
- 方便复用和分享

### ✅ 性能优化
- 使用 `loading="lazy"` 延迟加载
- 不影响文章主内容的加载速度

### ✅ 灵活性
- 可以使用任何 HTML/CSS/JS 功能
- 不受 Jekyll/Markdown 限制

## 示例

参考现有演示文件：
- `assets/demos/posts/2025-11-20-js-event-delegation-tab.html`

## 常见问题

### Q: 图片路径怎么写？
**A**: 使用绝对路径（以 `/` 开头，相对于网站根目录）：
```html
<!-- 正确 - Web 绝对路径 -->
<img src="/assets/images/posts/2025-11-20-slug/image.png">

<!-- 错误 - 相对路径在 iframe 中会找不到 -->
<img src="assets/images/posts/2025-11-20-slug/image.png">
<img src="../../images/posts/2025-11-20-slug/image.png">
```

**说明**：
- `/assets/...` 是 Web 绝对路径，相对于网站根目录
- 本地开发解析为：`http://localhost:4000/assets/...`
- 部署后解析为：`https://yourdomain.com/assets/...`
- 完全可移植，不影响其他人使用

### Q: 如何调整 iframe 高度？
**A**: 推荐使用自适应方案（方案一），iframe 会自动根据内容调整高度。如需固定高度，可手动设置 `height` 值。

### Q: 为什么自适应高度不生效？
**A**: 可能原因：
1. 同源策略限制：确保在 Jekyll 服务器环境下测试（`bundle exec jekyll serve`）
2. 内容加载时机：`onload` 在 iframe 加载完成后执行，如有异步内容可能需要调整

### Q: 演示文件能否使用外部资源？
**A**: 可以，但建议所有资源都放在项目内，确保离线可用。

### Q: 如何测试演示文件？
**A**: 直接用浏览器打开 HTML 文件，或通过 Jekyll 本地服务器访问。

## 更新日志

- 2025-11-20: 创建演示文件系统，添加 iframe 引用方案
