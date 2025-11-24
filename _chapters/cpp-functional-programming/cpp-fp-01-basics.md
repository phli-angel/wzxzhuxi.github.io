---
title: 第一章：函数式编程基础
book: cpp-functional-programming
order: 1
chapter_number: 1
difficulty: 基础
summary: 函数式编程核心思想、Lambda表达式与闭包、STL算法应用
permalink: /books/cpp-functional-programming/01-basics/
estimated_time: "45分钟"
github_exercises: "https://github.com/wzxzhuxi/cpp-functional-programming/tree/main/exercises"
github_solutions: "https://github.com/wzxzhuxi/cpp-functional-programming/tree/main/exercises/solutions"
---

## 什么是函数式编程

函数式编程（FP）核心思想：**把计算看作函数求值，避免状态变化和可变数据**。

### 核心特征

1. **函数是一等公民** - 函数可以作为参数、返回值、赋值给变量
2. **纯函数** - 相同输入永远产生相同输出，无副作用
3. **不可变数据** - 数据一旦创建就不再改变
4. **声明式** - 描述"做什么"而非"怎么做"
5. **组合性** - 小函数组合成复杂功能

### 命令式 vs 函数式

```cpp
// 命令式 - 告诉计算机怎么做
int sum = 0;
for (int i = 0; i < numbers.size(); i++) {
    if (numbers[i] % 2 == 0) {
        sum += numbers[i];
    }
}

// 函数式 - 告诉计算机做什么
auto sum = std::accumulate(
    numbers.begin(), numbers.end(), 0,
    [](int acc, int x) { return x % 2 == 0 ? acc + x : acc; }
);
```

第二种更清晰：求和、过滤偶数，一眼看穿。

## C++ 中的函数式特性

C++ 不是纯函数式语言，但从 C++11 开始支持够用的特性：

| 特性 | 版本 | 说明 |
|------|------|------|
| Lambda 表达式 | C++11 | 匿名函数 |
| `std::function` | C++11 | 通用函数包装器 |
| `constexpr` | C++11/14/17 | 编译期计算 |
| `std::optional` | C++17 | Maybe monad |
| `std::variant` | C++17 | Sum types |
| Ranges | C++20 | 惰性求值的函数式风格 |
| Concepts | C++20 | 类型约束 |

## Lambda 表达式

Lambda 是匿名函数，语法：

```cpp
[capture](parameters) -> return_type { body }
```

### 基本用法

```cpp
// 最简单的 lambda
auto add = [](int a, int b) { return a + b; };
std::cout << add(2, 3); // 5

// 自动推导返回类型
auto multiply = [](int a, int b) { return a * b; };

// 显式返回类型
auto divide = [](int a, int b) -> double {
    return static_cast<double>(a) / b;
};
```

### 捕获外部变量

```cpp
int x = 10;
int y = 20;

// 值捕获
auto f1 = [x]() { return x * 2; };

// 引用捕获
auto f2 = [&x]() { x += 5; };

// 捕获所有（值）
auto f3 = [=]() { return x + y; };

// 捕获所有（引用）
auto f4 = [&]() { x += y; };

// 混合捕获
auto f5 = [x, &y]() { return x + y; };
```

**警告**：引用捕获容易导致悬空引用，值捕获更安全。

### 实际应用

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> nums = {1, 2, 3, 4, 5, 6};

    // 过滤偶数
    std::vector<int> evens;
    std::copy_if(nums.begin(), nums.end(),
                 std::back_inserter(evens),
                 [](int x) { return x % 2 == 0; });

    // 转换：每个元素平方
    std::vector<int> squares;
    std::transform(nums.begin(), nums.end(),
                   std::back_inserter(squares),
                   [](int x) { return x * x; });

    // 排序：降序
    std::sort(nums.begin(), nums.end(),
              [](int a, int b) { return a > b; });

    return 0;
}
```

## 闭包

Lambda 捕获外部变量后形成**闭包** - 函数 + 环境。

```cpp
auto make_adder(int n) {
    // 返回的 lambda 捕获了 n
    return [n](int x) { return x + n; };
}

auto add5 = make_adder(5);
auto add10 = make_adder(10);

std::cout << add5(3);   // 8
std::cout << add10(3);  // 13
```

闭包记住了创建时的环境（`n` 的值）。

### 陷阱：引用捕获的生命周期

```cpp
auto make_bad_closure() {
    int x = 42;
    return [&x]() { return x; };  // 危险！x 已销毁
}

auto f = make_bad_closure();
f();  // 未定义行为 - x 已经不存在了
```

**规则**：返回 lambda 时，用值捕获，不用引用。

## std::function

`std::function` 是类型擦除的函数包装器，可以存储任何可调用对象：

```cpp
#include <functional>

std::function<int(int, int)> op;

// 存储 lambda
op = [](int a, int b) { return a + b; };
op(2, 3);  // 5

// 存储函数指针
int multiply(int a, int b) { return a * b; }
op = multiply;
op(2, 3);  // 6

// 存储函数对象
struct Divide {
    int operator()(int a, int b) const { return a / b; }
};
op = Divide{};
op(6, 3);  // 2
```

有性能开销（虚函数调用），但灵活。

## 实践建议

1. **默认用 auto lambda** - 除非需要存储或类型擦除
2. **优先值捕获** - 避免生命周期问题
3. **保持 lambda 简短** - 超过 3 行就抽成命名函数
4. **用 const** - 不改变捕获的值就加 `mutable`

---

## 完整代码示例

本节包含完整可运行示例，涵盖所有核心概念：

### 1. 基本 Lambda 用法

```cpp
void basic_lambdas() {
    std::cout << "=== Basic Lambdas ===\n";

    // 简单 lambda
    auto add = [](int a, int b) { return a + b; };
    std::cout << "add(2, 3) = " << add(2, 3) << "\n";

    // 显式返回类型
    auto divide = [](int a, int b) -> double {
        return static_cast<double>(a) / b;
    };
    std::cout << "divide(7, 2) = " << divide(7, 2) << "\n";

    // 无参数 lambda
    auto get_pi = []() { return 3.14159; };
    std::cout << "pi = " << get_pi() << "\n\n";
}
```

**要点**：
- 使用 `auto` 让编译器推导 lambda 类型
- 显式指定返回类型用于复杂计算或类型转换
- Lambda 本质是函数对象，可以像函数一样调用

### 2. 捕获外部变量

```cpp
void capture_examples() {
    std::cout << "=== Capture Examples ===\n";

    int x = 10;
    int y = 20;

    // 值捕获 - 创建副本
    auto f1 = [x]() { return x * 2; };
    std::cout << "f1() = " << f1() << " (x captured by value)\n";

    // 引用捕获 - 可以修改原变量
    auto f2 = [&x]() { x += 5; return x; };
    std::cout << "f2() = " << f2() << " (x modified via reference)\n";
    std::cout << "x is now: " << x << "\n";

    // 捕获所有（值）
    auto f3 = [=]() { return x + y; };
    std::cout << "f3() = " << f3() << " (captured all by value)\n";

    // 混合捕获
    auto f4 = [x, &y]() { y += x; return y; };
    std::cout << "f4() = " << f4() << " (x by value, y by ref)\n\n";
}
```

**要点**：
- `[x]` - 按值捕获 x（安全，但拷贝开销）
- `[&x]` - 按引用捕获 x（高效，但注意生命周期）
- `[=]` - 按值捕获所有外部变量
- `[&]` - 按引用捕获所有外部变量
- `[x, &y]` - 混合：x 按值，y 按引用

### 3. 闭包（Closure）

```cpp
// 工厂函数返回闭包
auto make_adder(int n) {
    return [n](int x) { return x + n; };
}

auto make_multiplier(int n) {
    return [n](int x) { return x * n; };
}

void closure_examples() {
    std::cout << "=== Closure Examples ===\n";

    // 创建不同的闭包实例
    auto add5 = make_adder(5);
    auto add10 = make_adder(10);
    auto times3 = make_multiplier(3);

    std::cout << "add5(7) = " << add5(7) << "\n";
    std::cout << "add10(7) = " << add10(7) << "\n";
    std::cout << "times3(7) = " << times3(7) << "\n\n";
}
```

**要点**：
- 闭包 = lambda + 捕获的环境
- 每次调用工厂函数创建独立的闭包
- 闭包记住创建时的状态（`n` 的值）

### 4. Lambda 用于 STL 算法

```cpp
void stl_algorithm_examples() {
    std::cout << "=== STL Algorithm Examples ===\n";

    std::vector<int> nums = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

    // 过滤偶数
    std::vector<int> evens;
    std::copy_if(nums.begin(), nums.end(),
                 std::back_inserter(evens),
                 [](int x) { return x % 2 == 0; });

    std::cout << "Evens: ";
    for (int n : evens) std::cout << n << " ";
    std::cout << "\n";

    // 平方变换
    std::vector<int> squares;
    std::transform(nums.begin(), nums.end(),
                   std::back_inserter(squares),
                   [](int x) { return x * x; });

    std::cout << "Squares: ";
    for (int n : squares) std::cout << n << " ";
    std::cout << "\n";

    // 求和
    auto sum = std::accumulate(nums.begin(), nums.end(), 0,
                               [](int acc, int x) { return acc + x; });
    std::cout << "Sum: " << sum << "\n";

    // 计数满足条件的元素
    auto count = std::count_if(nums.begin(), nums.end(),
                                [](int x) { return x > 5; });
    std::cout << "Count (>5): " << count << "\n\n";
}
```

**要点**：
- Lambda 作为谓词（predicate）用于 `copy_if`, `remove_if`
- Lambda 作为变换函数用于 `transform`
- Lambda 作为比较器用于 `sort`
- Lambda 作为累积函数用于 `accumulate`

### 5. std::function 示例

```cpp
void function_wrapper_examples() {
    std::cout << "=== std::function Examples ===\n";

    std::function<int(int, int)> op;

    // 存储 lambda
    op = [](int a, int b) { return a + b; };
    std::cout << "op(2, 3) with lambda: " << op(2, 3) << "\n";

    // 存储另一个 lambda
    op = [](int a, int b) { return a * b; };
    std::cout << "op(2, 3) with multiply: " << op(2, 3) << "\n";

    // 存储闭包
    int factor = 10;
    op = [factor](int a, int b) { return (a + b) * factor; };
    std::cout << "op(2, 3) with closure: " << op(2, 3) << "\n\n";
}
```

**要点**：
- `std::function` 可以存储任何匹配签名的可调用对象
- 有运行时开销（堆分配 + 虚函数调用）
- 适用于需要存储或传递不同类型函数的场景

### 6. 高阶函数示例

```cpp
// 自定义 filter 函数
template<typename T, typename Pred>
std::vector<T> filter(const std::vector<T>& vec, Pred pred) {
    std::vector<T> result;
    std::copy_if(vec.begin(), vec.end(),
                 std::back_inserter(result),
                 pred);
    return result;
}

// 自定义 map 函数
template<typename T, typename Func>
auto map(const std::vector<T>& vec, Func f) {
    using ReturnType = decltype(f(std::declval<T>()));
    std::vector<ReturnType> result;
    std::transform(vec.begin(), vec.end(),
                   std::back_inserter(result),
                   f);
    return result;
}

void higher_order_examples() {
    std::cout << "=== Higher-Order Function Examples ===\n";

    std::vector<int> nums = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

    // 使用自定义 filter
    auto evens = filter(nums, [](int x) { return x % 2 == 0; });
    std::cout << "Filtered evens: ";
    for (int n : evens) std::cout << n << " ";
    std::cout << "\n";

    // 使用自定义 map
    auto squares = map(nums, [](int x) { return x * x; });
    std::cout << "Mapped squares: ";
    for (int n : squares) std::cout << n << " ";
    std::cout << "\n";

    // 链式调用
    auto result = map(
        filter(nums, [](int x) { return x > 5; }),
        [](int x) { return x * 2; }
    );
    std::cout << "Filtered (>5) then doubled: ";
    for (int n : result) std::cout << n << " ";
    std::cout << "\n\n";
}
```

**要点**：
- 高阶函数接受函数作为参数或返回函数
- `filter` 和 `map` 是函数式编程的经典模式
- 可以链式组合多个操作，代码更简洁

### 运行完整示例

```cpp
int main() {
    basic_lambdas();
    capture_examples();
    closure_examples();
    stl_algorithm_examples();
    function_wrapper_examples();
    higher_order_examples();

    return 0;
}
```

### 构建并运行

```bash
# 克隆仓库
git clone https://github.com/wzxzhuxi/cpp-functional-programming.git
cd cpp-functional-programming

# 构建
mkdir build && cd build
cmake ..
make

# 运行示例
./bin/01_lambdas
```

**GitHub 完整源码**: [01_lambdas.cpp](https://github.com/wzxzhuxi/cpp-functional-programming/blob/main/examples/basic/01_lambdas.cpp)

### 示例输出

```
=== Basic Lambdas ===
add(2, 3) = 5
divide(7, 2) = 3.5
pi = 3.14159

=== Capture Examples ===
f1() = 20 (x captured by value)
f2() = 15 (x modified via reference)
x is now: 15
f3() = 35 (captured all by value)
f4() = 35 (x by value, y by ref)

=== Closure Examples ===
add5(7) = 12
add10(7) = 17
times3(7) = 21

=== STL Algorithm Examples ===
Evens: 2 4 6 8 10
Squares: 1 4 9 16 25 36 49 64 81 100
Sum: 55
Count (>5): 5

=== std::function Examples ===
op(2, 3) with lambda: 5
op(2, 3) with multiply: 6
op(2, 3) with closure: 50

=== Higher-Order Function Examples ===
Filtered evens: 2 4 6 8 10
Mapped squares: 1 4 9 16 25 36 49 64 81 100
Filtered (>5) then doubled: 12 14 16 18 20
```

### 关键要点总结

1. **Lambda 是匿名函数对象** - 编译器生成唯一类型
2. **值捕获更安全** - 避免悬空引用问题
3. **闭包捕获环境** - 函数 + 状态 = 强大抽象
4. **STL 算法的最佳搭档** - 简洁表达复杂逻辑
5. **性能无虞** - 编译器内联优化，零成本抽象

---

## 练习题

本章提供了配套的练习题，帮助你巩固所学知识：

1. 写一个 lambda，接收字符串数组，返回长度大于 5 的字符串
2. 实现 `make_multiplier(n)` 返回一个"乘以 n"的函数
3. 用 lambda 和 `std::sort` 对结构体数组按某字段排序

**[查看练习题 →](https://github.com/wzxzhuxi/cpp-functional-programming/blob/main/exercises/01_basics_exercises.cpp)**

练习题包含：
- 详细的问题描述
- 测试用例和预期输出
- 难度分级和提示

### 参考答案

⚠️ **建议先独立完成练习再查看答案**

**[查看参考答案 →](https://github.com/wzxzhuxi/cpp-functional-programming/blob/main/exercises/solutions/01_basics_solutions.cpp)**

答案包含：
- 完整的实现代码
- 详细的解题思路
- 常见错误分析
