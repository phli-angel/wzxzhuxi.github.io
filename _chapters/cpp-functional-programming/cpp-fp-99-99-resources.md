---
title: 学习资源与参考文献
book: cpp-functional-programming
order: 999
chapter_number: 99
section_type: resources
difficulty: 通用
summary: 推荐书籍、在线资源、工具库和社区链接
permalink: /books/cpp-functional-programming/resources/
estimated_time: "10分钟"
---

## 推荐书籍

### 函数式编程经典

1. **《Functional Programming in C++》** by Ivan Čukić
   - 本教程的主要参考书籍之一
   - 系统讲解 C++ 中的函数式编程技术
   - 包含大量实战案例

2. **《函数式编程图解》**（Illustrated Guide to Functional Programming）
   - 图解为主，易于理解
   - 适合初学者入门
   - 本教程参考书籍之一

3. **《Functional Programming Patterns in Scala and Clojure》** by Michael Bevilacqua-Linn
   - 虽然不是 C++，但概念通用
   - 学习函数式设计模式的好书

### C++ 进阶

4. **《Effective Modern C++》** by Scott Meyers
   - C++11/14 最佳实践
   - Lambda、移动语义、智能指针
   - 函数式编程的基础工具

5. **《C++ Templates: The Complete Guide》** (2nd Edition)
   - 模板元编程深入讲解
   - 高阶函数实现的理论基础

6. **《C++20 - The Complete Guide》** by Nicolai M. Josuttis
   - Ranges、Concepts 等新特性
   - 现代函数式 C++ 的利器

## 在线资源

### 官方文档

- [cppreference.com](https://en.cppreference.com/) - C++ 标准库权威参考
- [C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines) - C++ 编程规范

### 教程与博客

- [Bartosz Milewski's Blog](https://bartoszmilewski.com/) - 范畴论与函数式编程
- [Fluent C++](https://www.fluentcpp.com/) - 现代 C++ 技巧
- [ModernesC++](https://www.modernescpp.com/) - 现代 C++ 特性讲解
- [Jonathan Boccara's Blog](https://www.fluentcpp.com/) - Expressive C++ 系列

### 视频课程

- [CppCon YouTube Channel](https://www.youtube.com/user/CppCon) - C++ 大会演讲
  - 搜索关键词："functional programming", "ranges", "concepts"
- [C++ Weekly by Jason Turner](https://www.youtube.com/c/lefticus1) - 每周 C++ 技巧

### 交互式学习

- [Compiler Explorer (Godbolt)](https://godbolt.org/) - 在线查看编译器生成的汇编代码
- [C++ Insights](https://cppinsights.io/) - 查看编译器如何转换你的代码（lambda、模板展开等）
- [Quick Bench](https://quick-bench.com/) - 在线性能基准测试

## 函数式 C++ 库

### 1. Range-v3

功能强大的 Ranges 库，C++20 Ranges 的前身。

- **GitHub**: [https://github.com/ericniebler/range-v3](https://github.com/ericniebler/range-v3)
- **特性**: 惰性求值、管道操作、视图组合
- **使用场景**: 函数式数据转换

```cpp
#include <range/v3/all.hpp>
using namespace ranges;

auto result = vec
    | views::filter([](int x) { return x % 2 == 0; })
    | views::transform([](int x) { return x * x; })
    | views::take(5);
```

### 2. Boost.Hana

编译期元编程库，类型级别的函数式编程。

- **官网**: [https://www.boost.org/libs/hana/](https://www.boost.org/libs/hana/)
- **特性**: 类型计算、编译期容器、元函数组合
- **使用场景**: 模板元编程、编译期优化

### 3. FunctionalPlus

实用的函数式编程工具库。

- **GitHub**: [https://github.com/Dobiasd/FunctionalPlus](https://github.com/Dobiasd/FunctionalPlus)
- **特性**: map、filter、fold、curry、compose
- **使用场景**: 日常函数式编程工具

```cpp
#include <fplus/fplus.hpp>

auto result = fplus::transform(
    fplus::square<int>,
    fplus::keep_if(fplus::is_even<int>, vec)
);
```

### 4. Cat (Category Theory)

范畴论抽象库。

- **GitHub**: [https://github.com/awgn/cat](https://github.com/awgn/cat)
- **特性**: Functor、Applicative、Monad、Arrow
- **使用场景**: 高级抽象、范畴论实践

### 5. Immer

不可变数据结构库，持久化数据结构实现。

- **GitHub**: [https://github.com/arximboldi/immer](https://github.com/arximboldi/immer)
- **特性**: Vector、Map、Set 的不可变版本，结构共享
- **使用场景**: 函数式数据结构、时间旅行

## 相关主题

### 并发与异步

函数式编程与并发编程天然契合：

- **RxCpp**: Reactive Extensions for C++
  - [GitHub](https://github.com/ReactiveX/RxCpp)
  - 异步数据流的函数式处理

- **Seastar**: 高性能异步框架
  - [GitHub](https://github.com/scylladb/seastar)
  - Future/Promise 模式

### 类型系统

- **C++20 Concepts**: 类型约束，更安全的泛型编程
- **Optional/Expected**: 类型安全的错误处理

### 性能优化

- **Copy Elision & RVO**: 编译器优化，消除不必要的拷贝
- **Move Semantics**: 零成本抽象的关键
- **constexpr**: 编译期计算，零运行时开销

## 社区与讨论

### 论坛

- [Stack Overflow - C++ Tag](https://stackoverflow.com/questions/tagged/c%2b%2b)
- [Reddit r/cpp](https://www.reddit.com/r/cpp/)
- [C++ Slack Community](https://cpplang.slack.com/)

### 会议

- **CppCon** - 北美最大 C++ 会议
- **Meeting C++** - 欧洲 C++ 会议
- **C++ Now** - 面向库开发者的会议
- **ACCU** - 英国 C++ 用户组会议

### 中文社区

- [CppCon 中文字幕组](https://github.com/CppCon-Chinese-Subtitles)
- [现代 C++ 教程](https://changkun.de/modern-cpp/)
- [C++ 中文论坛](https://www.cplusplus.com/)

## 工具推荐

### 编译器

- **GCC** - GNU Compiler Collection (推荐 11+)
- **Clang** - LLVM 前端 (推荐 14+)
- **MSVC** - Microsoft Visual C++ (2019+)

### 构建系统

- **CMake** - 跨平台构建系统
- **Meson** - 现代化构建工具
- **Bazel** - Google 的大规模构建系统

### 静态分析

- **Clang-Tidy** - 代码检查工具
- **Cppcheck** - 静态分析工具
- **Coverity** - 商业静态分析

### 调试工具

- **GDB** - GNU 调试器
- **LLDB** - LLVM 调试器
- **Valgrind** - 内存泄漏检测
- **AddressSanitizer** - 内存错误检测

## 练习平台

- [LeetCode](https://leetcode.com/) - 算法题，可用 C++ 函数式风格解答
- [HackerRank](https://www.hackerrank.com/) - 编程挑战
- [Exercism C++ Track](https://exercism.org/tracks/cpp) - C++ 练习题库

## 论文与学术资源

### 经典论文

1. **"Why Functional Programming Matters"** by John Hughes
   - 函数式编程为什么重要的经典论文

2. **"The Essence of Functional Programming"** by Philip Wadler
   - Monad 概念的清晰讲解

3. **"Making Data Structures Persistent"** by James R. Driscoll et al.
   - 持久化数据结构的理论基础

### 学术期刊

- **Journal of Functional Programming** (Cambridge)
- **ICFP** - International Conference on Functional Programming

## 贡献本教程

本教程是开源项目，欢迎贡献：

- **GitHub 仓库**: [wzxzhuxi/cpp-functional-programming](https://github.com/wzxzhuxi/cpp-functional-programming)
- **提交 Issue**: 报告错误、建议改进
- **Pull Request**: 贡献代码、修正文档
- **讨论**: 分享你的学习心得

## 联系方式

- **作者**: Zhuxi
- **博客**: [A_Bit_S Blog](https://wzxzhuxi.github.io/)
- **GitHub**: [@wzxzhuxi](https://github.com/wzxzhuxi)

---

**开始学习之旅**:

← [返回书籍首页](/books/cpp-functional-programming/)

← [从第一章开始](/books/cpp-functional-programming/01-basics/)
