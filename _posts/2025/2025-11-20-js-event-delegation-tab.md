---
layout: post
title: "用事件委托实现高效 Tab 切换组件（电商特价页实战）"
author: lele
date: 2025-11-20
categories: [前端开发]
tags: [JavaScript, 事件委托, DOM操作, 电商组件]
excerpt: >
  结合电商特价页场景，用事件委托实现 Tab 切换功能，减少事件监听数量，适配动态元素，提升页面性能。
cover: /assets/images/posts/2025-11-20-js-event-delegation-tab/tab-discount.png
---

## 需求说明
实现电商平台「今日特价」模块的 Tab 切换功能：
- 点击「精选」「美食」「百货」等 Tab 标签，标签自动高亮；
- 同步切换对应的商品内容区域；
- 要求代码简洁高效，支持动态新增 Tab 无需额外绑定事件。

## 在线演示

<div style="margin: 20px 0;">
    <iframe
        id="demo-iframe-2025-11-20"
        src="/assets/demos/posts/2025-11-20-js-event-delegation-tab.html"
        style="width: 100%; border: 2px solid #333; border-radius: 4px;"
        frameborder="0"
        scrolling="no"
        onload="this.style.height=(this.contentWindow.document.body.scrollHeight)+'px'">
    </iframe>
</div>

**💡 提示**：点击 Tab 按钮体验切换效果。这是完全独立的 HTML 演示，展示了事件委托的实际运行效果。

## 技术要点

### 1. 事件委托优势
- **减少事件监听器数量**：只在父元素 `ul` 上绑定一个事件，而不是给每个 `a` 标签都绑定
- **支持动态元素**：新增的 Tab 无需额外绑定事件，自动响应点击
- **性能优化**：降低内存占用，提升页面性能

### 2. 核心实现逻辑
1. 在父容器上监听点击事件
2. 通过 `e.target` 判断点击的是否为目标元素（`a` 标签）
3. 使用 `data-id` 属性存储 Tab 索引
4. 通过 `classList` API 切换激活状态

### 3. 关键代码说明
```javascript
// 将字符串转为数字
const tabIndex = +e.target.dataset.id;

// nth-child 选择器索引从 1 开始
document.querySelector(`.right .rig:nth-child(${tabIndex + 1})`)
```

## 核心 JavaScript 实现

```javascript
// 获取 Tab 导航父容器（事件委托目标）
const tabNavParent = document.querySelector('.nav ul');

// 绑定事件到父容器，实现事件委托
tabNavParent.addEventListener('click', function(e) {
    // 1. 判断事件源是否为 Tab 标签（a 标签）
    if (e.target.tagName === 'A') {
        // 阻止 a 标签默认跳转行为
        e.preventDefault();

        // 2. 移除所有 Tab 标签的激活状态
        document.querySelectorAll('.nav a').forEach(tab => {
            tab.classList.remove('active');
        });

        // 3. 给当前点击的标签添加激活状态
        e.target.classList.add('active');

        // 4. 获取当前 Tab 对应的索引（从 data-id 属性读取）
        const tabIndex = +e.target.dataset.id; // + 号将字符串转为数字

        // 5. 移除所有内容区域的激活状态
        document.querySelectorAll('.right .rig').forEach(content => {
            content.classList.remove('active');
        });

        // 6. 激活当前 Tab 对应的内容区域（nth-child 索引从 1 开始，需+1）
        document.querySelector(`.right .rig:nth-child(${tabIndex + 1})`).classList.add('active');
    }
});
```

## 完整代码（HTML + CSS）

### 静态页面结构

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>今日特价 - 电商特价页</title>
    <style>
        /* 基础样式 */
        .box {
            margin: 0 auto;
            width: 800px;
            height: 400px;
            background-color: rgb(251, 249, 249);
        }

        .nav {
            margin: 0 auto;
            width: 800px;
            height: 50px;
        }

        .nav h3 {
            margin-left: 30px;
            float: left;
        }

        .nav ul {
            margin-top: 20px;
            margin-right: 30px;
            float: right;
            width: 400px;
            height: 50px;
            list-style: none;
        }

        .nav a {
            text-decoration: none;
            color: black;
            margin-left: 40px;
            float: left;
            line-height: 35px;
            width: 35px;
            height: 35px;
        }

        /* 激活状态样式 */
        .nav .active {
            color: red;
        }

        /* 内容区域样式 */
        .banner {
            margin-top: 20px;
            position: relative;
            width: 800px;
            height: 500px;
        }

        .left {
            position: absolute;
            left: 0;
            width: 200px;
            height: 330px;
        }

        .left img {
            width: 200px;
            height: 200px;
        }

        .left p {
            display: block;
            margin-left: 30px;
        }

        .left p[color="red"] {
            color: red;
        }

        .right {
            position: absolute;
            right: 0;
            padding-left: 30px;
            width: 570px;
            height: 300px;
        }

        .right .rig {
            display: none;
        }

        .right .active {
            display: block;
        }

        .right img {
            width: 120px;
            height: 120px;
            margin-right: 10px;
        }
    </style>
</head>
<body>
    <div class="box">
        <!-- Tab 导航区域 -->
        <div class="nav">
            <h3>今日特价</h3>
            <ul>
                <li><a href="" class="active" data-id="0">精选</a></li>
                <li><a href="" data-id="1">美食</a></li>
                <li><a href="" data-id="2">百货</a></li>
                <li><a href="" data-id="3">个护</a></li>
                <li><a href="" data-id="4">预告</a></li>
            </ul>
        </div>

        <!-- 商品内容区域 -->
        <div class="banner">
            <div class="left">
                <img src="/assets/images/posts/2025-11-20-js-event-delegation-tab/tab-discount.png" alt="美白祛斑产品">
                <span>
                    <p>【10片】美白祛斑</p>
                    <p color="red">9.9元</p>
                    <p>已抢150件</p>
                </span>
            </div>
            <div class="right">
                <div class="rig active">
                    <img src="/assets/images/posts/2025-11-20-js-event-delegation-tab/tab-drink.png" alt="饮料组合">饮料组合￥9.9
                    <img src="/assets/images/posts/2025-11-20-js-event-delegation-tab/tab-drink.png" alt="饮料组合">饮料组合￥8.8
                    <img src="/assets/images/posts/2025-11-20-js-event-delegation-tab/tab-drink.png" alt="饮料组合">饮料组合￥7.7
                    <img src="/assets/images/posts/2025-11-20-js-event-delegation-tab/tab-drink.png" alt="饮料组合">饮料组合￥6.6
                </div>
                <div class="rig">
                    <img src="/assets/images/posts/2025-11-20-js-event-delegation-tab/tab-snack.png" alt="零食套盒">零食套盒￥9.9
                    <img src="/assets/images/posts/2025-11-20-js-event-delegation-tab/tab-snack.png" alt="零食套盒">零食套盒￥8.8
                    <img src="/assets/images/posts/2025-11-20-js-event-delegation-tab/tab-snack.png" alt="零食套盒">零食套盒￥7.7
                    <img src="/assets/images/posts/2025-11-20-js-event-delegation-tab/tab-snack.png" alt="零食套盒">零食套盒￥6.6
                </div>
                <div class="rig">
                    <img src="/assets/images/posts/2025-11-20-js-event-delegation-tab/tab-juice.png" alt="果汁套盒">果汁套盒￥9.9
                    <img src="/assets/images/posts/2025-11-20-js-event-delegation-tab/tab-juice.png" alt="果汁套盒">果汁套盒￥8.8
                    <img src="/assets/images/posts/2025-11-20-js-event-delegation-tab/tab-juice.png" alt="果汁套盒">果汁套盒￥7.7
                    <img src="/assets/images/posts/2025-11-20-js-event-delegation-tab/tab-juice.png" alt="果汁套盒">果汁套盒￥6.6
                </div>
                <div class="rig">
                    <img src="/assets/images/posts/2025-11-20-js-event-delegation-tab/tab-beer.png" alt="啤酒套盒">啤酒套盒￥9.9
                    <img src="/assets/images/posts/2025-11-20-js-event-delegation-tab/tab-beer.png" alt="啤酒套盒">啤酒套盒￥8.8
                    <img src="/assets/images/posts/2025-11-20-js-event-delegation-tab/tab-beer.png" alt="啤酒套盒">啤酒套盒￥7.7
                    <img src="/assets/images/posts/2025-11-20-js-event-delegation-tab/tab-beer.png" alt="啤酒套盒">啤酒套盒￥6.6
                </div>
                <div class="rig">
                    <img src="/assets/images/posts/2025-11-20-js-event-delegation-tab/tab-pen.png" alt="荧光笔">莫兰迪色荧光笔￥9.9
                    <img src="/assets/images/posts/2025-11-20-js-event-delegation-tab/tab-pen.png" alt="荧光笔">莫兰迪色荧光笔￥8.8
                    <img src="/assets/images/posts/2025-11-20-js-event-delegation-tab/tab-pen.png" alt="荧光笔">莫兰迪色荧光笔￥7.7
                    <img src="/assets/images/posts/2025-11-20-js-event-delegation-tab/tab-pen.png" alt="荧光笔">莫兰迪色荧光笔￥6.6
                </div>
            </div>
        </div>
    </div>
</body>
</html>
```

## 总结

事件委托是 JavaScript 中的重要优化技术，特别适用于：
- 列表项的点击处理
- Tab 切换、下拉菜单等交互组件
- 动态生成的内容

通过事件委托，我们用更少的代码实现了更高效的 Tab 切换功能，这正是现代前端开发应该掌握的核心技能。
