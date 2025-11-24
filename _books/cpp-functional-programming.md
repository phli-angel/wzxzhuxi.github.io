---
title: C++ 函数式编程
slug: cpp-functional-programming
description: 以现代 C++ 为起点，循序渐进拆解函数式思想、语法特性与工程实践，帮助开发者写出更安全、更可组合的代码。主要参考书籍《Functional Programming in C++》与《函数式编程图解》，并结合工程一线案例做本土化解读。
cover:
order: 2
author: zhuxi
github_repo: "https://github.com/wzxzhuxi/cpp-functional-programming"
---

## 关于本教程

函数式范式并不是 FP 语言的专属。本教程立足 C++17/20/23，结合我们在嵌入式与后端项目中的经验，讲解如何在"非纯"语言中引入纯函数、不可变数据结构与高阶抽象。

**Talk is cheap. Show me the code.** 这不是纸上谈兵的理论课——每个概念都配有完整可运行的代码示例和实战练习。

## 学习路径

### 基础篇（第 1-3 章）
- **第一章：基础概念** - Lambda 表达式、闭包、std::function
- **第二章：不可变性** - const 正确性、不可变数据结构、持久化数据结构
- **第三章：纯函数** - 函数纯度、副作用隔离、引用透明性

### 核心篇（第 4-6 章）
- **第四章：高阶函数** - map/filter/reduce、函数柯里化、偏应用
- **第五章：函数组合** - pipe/compose 操作符、点自由风格
- **第六章：代数数据类型** - Sum/Product types、std::variant、模式匹配

### 进阶篇（第 7-8 章）
- **第七章：Monad 模式** - Maybe、Result/Either、IO、List monads
- **第八章：高级模式** - 惰性求值、记忆化、蹦床函数、延续、透镜、Y组合子

## 如何使用

1. **📖 阅读理论**：每章的"概念"部分讲解核心思想和原理
2. **💻 运行示例**：查看完整可编译的代码示例，理解实际应用
3. **✏️ 完成练习**：通过实战巩固所学，从 GitHub 下载练习模板
4. **✅ 对照答案**：完成练习后查看参考答案和解题思路

## 配套代码仓库

📥 **完整代码与练习**: [GitHub - cpp-functional-programming](https://github.com/wzxzhuxi/cpp-functional-programming)

仓库包含：
- 8 个完整的示例程序（可编译运行）
- 16 套练习题与参考答案
- CMake 构建配置
- 环境配置指南

**环境要求**: C++17/20, GCC 10+/Clang 12+/MSVC 2019+, CMake 3.15+

## 核心原则

- **Immutability** - 默认不可变
- **Pure Functions** - 同样输入 → 同样输出
- **Composition** - 小函数组合成复杂逻辑
- **Declarative** - 说"要什么"，不是"怎么做"
- **Type Safety** - 类型系统是你的朋友

## 贡献与反馈

发现问题或有改进建议？欢迎在 [GitHub Issues](https://github.com/wzxzhuxi/cpp-functional-programming/issues) 提出。
