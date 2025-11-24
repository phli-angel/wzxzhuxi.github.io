---
title: 第六章:代数数据类型
book: cpp-functional-programming
order: 6
chapter_number: 6
difficulty: 进阶
summary: 积类型、和类型、std::variant、std::optional、模式匹配
permalink: /books/cpp-functional-programming/06-algebraic-types/
estimated_time: "60分钟"
github_exercises: "https://github.com/wzxzhuxi/cpp-functional-programming/tree/main/exercises"
github_solutions: "https://github.com/wzxzhuxi/cpp-functional-programming/tree/main/exercises/solutions"
---

## 说人话:代数数据类型是什么?

代数数据类型(ADT)不是什么高深的数学概念。**它就是用类型系统精确表达数据结构的方式**。

分为两大类:
1. **积类型(Product Types)**:"AND"关系,同时拥有多个字段(struct、tuple)
2. **和类型(Sum Types)**:"OR"关系,多个可能性之一(variant、optional)

**核心思想**:让非法状态无法表示。

## 1. 积类型(Product Types)

积类型表示"AND"关系:同时拥有多个字段。

### 为什么叫"积"?

```cpp
struct Point {
    int x;  // 假设 int 有 N 种可能
    int y;  // 假设 int 有 N 种可能
};
// Point 的可能值数量 = N × N(笛卡尔积)
```

### Struct

```cpp
struct Person {
    std::string name;
    int age;
    double height;
};

Person alice{"Alice", 25, 1.65};
```

### Tuple

```cpp
std::tuple<int, std::string, double> person{25, "Alice", 1.65};

// 结构化绑定(C++17)
auto [age, name, height] = person;
std::cout << name << " is " << age << " years old\n";
```

### Pair

```cpp
std::pair<std::string, int> score{"Alice", 95};
std::cout << score.first << " scored " << score.second << "\n";
```

## 2. 和类型(Sum Types)

和类型表示"OR"关系:多个可能性之一。

### 为什么叫"和"?

```cpp
std::variant<int, bool> value;
// value 的可能值数量 = int 的数量 + bool 的数量(2)
```

### std::variant

```cpp
std::variant<int, double, std::string> value;

value = 42;           // 现在是 int
value = 3.14;         // 现在是 double
value = "hello";      // 现在是 string

// 访问
std::visit([](auto&& arg) {
    std::cout << arg << "\n";
}, value);
```

### std::optional

```cpp
std::optional<int> maybe_value;

maybe_value = 42;           // 有值
maybe_value = std::nullopt; // 无值

if (maybe_value) {
    std::cout << *maybe_value;
}

// 或使用 value_or
int val = maybe_value.value_or(0);
```

## 3. 模式匹配

C++ 没有原生模式匹配,但可以用 `std::visit` 模拟。

### 基础访问

```cpp
std::variant<int, std::string> v = 42;

std::visit([](auto&& arg) {
    using T = std::decay_t<decltype(arg)>;
    if constexpr (std::is_same_v<T, int>) {
        std::cout << "Integer: " << arg << "\n";
    } else if constexpr (std::is_same_v<T, std::string>) {
        std::cout << "String: " << arg << "\n";
    }
}, v);
```

### Overload 模式(推荐)

```cpp
template<class... Ts>
struct overload : Ts... {
    using Ts::operator()...;
};

template<class... Ts>
overload(Ts...) -> overload<Ts...>;

// 使用
std::visit(overload{
    [](int i) { std::cout << "Int: " << i << "\n"; },
    [](const std::string& s) { std::cout << "String: " << s << "\n"; }
}, v);
```

## 4. Result/Either 类型

用于函数式错误处理。

### 实现

```cpp
template<typename T, typename E>
struct Result {
    std::variant<T, E> data;

    bool is_ok() const {
        return std::holds_alternative<T>(data);
    }

    const T& unwrap() const {
        return std::get<T>(data);
    }

    const E& unwrap_err() const {
        return std::get<E>(data);
    }

    static Result ok(T value) {
        return Result{std::variant<T, E>(std::in_place_index<0>, value)};
    }

    static Result err(E error) {
        return Result{std::variant<T, E>(std::in_place_index<1>, error)};
    }
};
```

### 使用

```cpp
Result<int, std::string> divide(int a, int b) {
    if (b == 0) {
        return Result<int, std::string>::err("Division by zero");
    }
    return Result<int, std::string>::ok(a / b);
}

auto result = divide(10, 2);
if (result.is_ok()) {
    std::cout << result.unwrap();
} else {
    std::cout << "Error: " << result.unwrap_err();
}
```

## 5. 递归类型

### 表达式树

```cpp
struct Expr;
using ExprPtr = std::shared_ptr<Expr>;

struct Literal { int value; };
struct Add { ExprPtr left, right; };
struct Mul { ExprPtr left, right; };

struct Expr {
    std::variant<Literal, Add, Mul> data;
};

// 求值
int eval(const Expr& expr) {
    return std::visit(overload{
        [](const Literal& lit) { return lit.value; },
        [](const Add& add) { return eval(*add.left) + eval(*add.right); },
        [](const Mul& mul) { return eval(*mul.left) * eval(*mul.right); }
    }, expr.data);
}
```

## 6. 实际应用

### 6.1 状态机

```cpp
struct Idle {};
struct Running { int progress; };
struct Completed { std::string result; };
struct Failed { std::string error; };

using State = std::variant<Idle, Running, Completed, Failed>;

void handle_state(const State& state) {
    std::visit(overload{
        [](const Idle&) { std::cout << "Waiting...\n"; },
        [](const Running& r) { std::cout << "Progress: " << r.progress << "%\n"; },
        [](const Completed& c) { std::cout << "Done: " << c.result << "\n"; },
        [](const Failed& f) { std::cout << "Error: " << f.error << "\n"; }
    }, state);
}
```

### 6.2 配置解析

```cpp
struct Config {
    std::variant<int, double, std::string, bool> value;

    template<typename T>
    std::optional<T> get() const {
        if (std::holds_alternative<T>(value)) {
            return std::get<T>(value);
        }
        return std::nullopt;
    }
};
```

### 6.3 API 响应

```cpp
struct Success { std::string data; };
struct NotFound { std::string resource; };
struct ServerError { int code; std::string message; };

using ApiResponse = std::variant<Success, NotFound, ServerError>;

void handle_response(const ApiResponse& resp) {
    std::visit(overload{
        [](const Success& s) {
            std::cout << "Data: " << s.data << "\n";
        },
        [](const NotFound& nf) {
            std::cout << "404: " << nf.resource << " not found\n";
        },
        [](const ServerError& se) {
            std::cout << "Error " << se.code << ": " << se.message << "\n";
        }
    }, resp);
}
```

## 7. 优势

### 类型安全

使用 variant 而非 enum + union,编译器保证类型安全。

### 穷举检查

编译器会警告未处理的情况。

### 不可能状态

❌ **使用布尔标记(允许非法状态)**:
```cpp
struct Task {
    bool is_running;
    bool is_completed;
    // 可能同时 is_running=true && is_completed=true(非法！)
};
```

✅ **使用 variant(只允许合法状态)**:
```cpp
using TaskState = std::variant<Idle, Running, Completed>;
// 不可能同时处于多个状态
```

---

## 最佳实践

1. **优先 variant 而非 void* 或 union**:类型安全
2. **使用 optional 而非空指针**:明确表达"可能没有值"
3. **Result 类型而非异常**:显式错误处理
4. **不可变 ADT**:所有字段 const
5. **使用 overload 模式**:简化 visit 代码

## 为什么代数数据类型有用?

### 1. 类型安全
编译器保证正确性,不会访问错误类型。

### 2. 表达力
精确建模领域,不可能状态无法表示。

### 3. 可组合性
积类型 + 和类型 = 任意复杂结构。

### 4. 模式匹配
优雅处理不同情况,避免 if-else 地狱。

## 总结

代数数据类型提供:
- **积类型(struct/tuple)**:AND 关系,组合多个值
- **和类型(variant/optional)**:OR 关系,多个可能性之一
- **模式匹配(visit + overload)**:优雅访问和类型
- **类型安全**:编译器保证正确性
- **表达力**:精确建模,避免非法状态

掌握 ADT,你的代码会更安全、更清晰、更易维护。

---

## 完整代码示例

本节包含完整可运行示例,涵盖所有核心概念:

### Overload 模式定义

```cpp
#include <cmath>
#include <iostream>
#include <memory>
#include <optional>
#include <string>
#include <variant>
#include <vector>

// ============================================
// Overload pattern for visit
// ============================================

template<class... Ts>
struct overload : Ts... { using Ts::operator()...; };

template<class... Ts>
overload(Ts...) -> overload<Ts...>;
```

**要点**:
- Overload 模式利用继承和可变参数模板
- 使用 `using Ts::operator()...` 展开所有 lambda
- C++17 推导指南简化使用

### 1. 积类型示例

```cpp
// ============================================
// 1. Product Types
// ============================================

void product_types_demo() {
    std::cout << "=== Product Types ===\n";

    // Tuple
    std::tuple<int, std::string, double> person{25, "Alice", 1.65};
    auto [age, name, height] = person;
    std::cout << name << " is " << age << " years old, " << height << "m tall\n";

    // Pair
    std::pair<std::string, int> score{"Bob", 95};
    std::cout << score.first << " scored " << score.second << "\n\n";
}
```

**要点**:
- Tuple 可包含不同类型的多个值
- 结构化绑定(C++17)简化访问
- Pair 是特殊的 2 元组
- 积类型表示笛卡尔积:所有字段组合

### 2. 和类型(variant)示例

```cpp
// ============================================
// 2. Sum Types - variant
// ============================================

void sum_types_demo() {
    std::cout << "=== Sum Types (variant) ===\n";

    std::variant<int, double, std::string> value;

    value = 42;
    std::visit([](auto&& arg) { std::cout << "Value: " << arg << "\n"; }, value);

    value = 3.14;
    std::visit([](auto&& arg) { std::cout << "Value: " << arg << "\n"; }, value);

    value = "hello";
    std::visit([](auto&& arg) { std::cout << "Value: " << arg << "\n"; }, value);
    std::cout << "\n";
}
```

**要点**:
- Variant 一次只能持有一个类型的值
- 类型安全,不能访问错误类型
- `std::visit` 对所有可能类型应用操作
- 和类型表示"或":int 或 double 或 string

### 3. Optional 示例

```cpp
// ============================================
// 3. Optional
// ============================================

std::optional<int> find_value(const std::vector<int>& vec, int target) {
    for (int v : vec) {
        if (v == target) return v;
    }
    return std::nullopt;
}

void optional_demo() {
    std::cout << "=== Optional ===\n";

    std::vector<int> nums{1, 2, 3, 4, 5};

    auto result1 = find_value(nums, 3);
    if (result1) {
        std::cout << "Found: " << *result1 << "\n";
    }

    auto result2 = find_value(nums, 10);
    if (!result2) {
        std::cout << "Not found: 10\n";
    }
    std::cout << "\n";
}
```

**要点**:
- Optional 表示"可能没有值"
- 比空指针更安全,类型明确
- 用 `if (optional)` 检查是否有值
- 用 `*optional` 或 `value()` 获取值

### 4. 模式匹配示例

```cpp
// ============================================
// 4. Pattern Matching with overload
// ============================================

void pattern_matching_demo() {
    std::cout << "=== Pattern Matching ===\n";

    std::variant<int, double, std::string> v = 42;

    std::visit(overload{
        [](int i) { std::cout << "Integer: " << i << "\n"; },
        [](double d) { std::cout << "Double: " << d << "\n"; },
        [](const std::string& s) { std::cout << "String: " << s << "\n"; }
    }, v);

    v = 3.14159;
    std::visit(overload{
        [](int i) { std::cout << "Integer: " << i << "\n"; },
        [](double d) { std::cout << "Double: " << d << "\n"; },
        [](const std::string& s) { std::cout << "String: " << s << "\n"; }
    }, v);

    v = "test";
    std::visit(overload{
        [](int i) { std::cout << "Integer: " << i << "\n"; },
        [](double d) { std::cout << "Double: " << d << "\n"; },
        [](const std::string& s) { std::cout << "String: " << s << "\n"; }
    }, v);
    std::cout << "\n";
}
```

**要点**:
- Overload 模式让 visit 更清晰
- 每种类型有专门的处理逻辑
- 编译器检查是否处理所有类型
- 比 if-else 或 switch 更安全

### 5. Result 类型示例

```cpp
// ============================================
// 5. Result Type
// ============================================

template<typename T, typename E>
struct Result {
    std::variant<T, E> data;

    bool is_ok() const { return std::holds_alternative<T>(data); }
    const T& unwrap() const { return std::get<T>(data); }
    const E& unwrap_err() const { return std::get<E>(data); }

    static Result ok(T value) { return Result{std::variant<T, E>(std::in_place_index<0>, value)}; }
    static Result err(E error) { return Result{std::variant<T, E>(std::in_place_index<1>, error)}; }
};

Result<int, std::string> safe_divide(int a, int b) {
    if (b == 0) return Result<int, std::string>::err("Division by zero");
    return Result<int, std::string>::ok(a / b);
}

Result<double, std::string> safe_sqrt(double x) {
    if (x < 0) return Result<double, std::string>::err("Negative sqrt");
    return Result<double, std::string>::ok(std::sqrt(x));
}

void result_type_demo() {
    std::cout << "=== Result Type ===\n";

    auto r1 = safe_divide(10, 2);
    if (r1.is_ok()) {
        std::cout << "10 / 2 = " << r1.unwrap() << "\n";
    } else {
        std::cout << "Error: " << r1.unwrap_err() << "\n";
    }

    auto r2 = safe_divide(10, 0);
    if (!r2.is_ok()) {
        std::cout << "10 / 0 error: " << r2.unwrap_err() << "\n";
    }

    auto r3 = safe_sqrt(16.0);
    if (r3.is_ok()) {
        std::cout << "sqrt(16) = " << r3.unwrap() << "\n";
    }

    auto r4 = safe_sqrt(-4.0);
    if (!r4.is_ok()) {
        std::cout << "sqrt(-4) error: " << r4.unwrap_err() << "\n";
    }
    std::cout << "\n";
}
```

**要点**:
- Result 明确表达成功或失败
- 比异常更显式,错误类型在签名中
- `ok` 包装成功值,`err` 包装错误
- 强制处理错误,不能忽略

### 6. 状态机示例

```cpp
// ============================================
// 6. State Machine
// ============================================

struct Idle {};
struct Running { int progress; };
struct Completed { std::string result; };
struct Failed { std::string error; };

using State = std::variant<Idle, Running, Completed, Failed>;

void handle_state(const State& state) {
    std::visit(overload{
        [](const Idle&) { std::cout << "State: Idle\n"; },
        [](const Running& r) { std::cout << "State: Running (" << r.progress << "%)\n"; },
        [](const Completed& c) { std::cout << "State: Completed - " << c.result << "\n"; },
        [](const Failed& f) { std::cout << "State: Failed - " << f.error << "\n"; }
    }, state);
}

void state_machine_demo() {
    std::cout << "=== State Machine ===\n";

    std::vector<State> states{
        Idle{},
        Running{25},
        Running{50},
        Running{75},
        Completed{"Success!"}
    };

    for (const auto& s : states) {
        handle_state(s);
    }

    // 错误状态
    State failed = Failed{"Network timeout"};
    handle_state(failed);
    std::cout << "\n";
}
```

**要点**:
- 用 variant 建模状态机,只能处于一个状态
- 每个状态可携带不同数据
- 不可能处于非法状态组合
- 模式匹配处理所有状态转换

### 7. 表达式树示例

```cpp
// ============================================
// 7. Expression Tree (Recursive ADT)
// ============================================

struct Expr;
using ExprPtr = std::shared_ptr<Expr>;

struct Literal { int value; };
struct Add { ExprPtr left, right; };
struct Mul { ExprPtr left, right; };
struct Sub { ExprPtr left, right; };

struct Expr {
    std::variant<Literal, Add, Mul, Sub> data;

    Expr(Literal lit) : data(lit) {}
    Expr(Add add) : data(add) {}
    Expr(Mul mul) : data(mul) {}
    Expr(Sub sub) : data(sub) {}
};

ExprPtr lit(int value) {
    return std::make_shared<Expr>(Literal{value});
}

ExprPtr add(ExprPtr left, ExprPtr right) {
    return std::make_shared<Expr>(Add{left, right});
}

ExprPtr mul(ExprPtr left, ExprPtr right) {
    return std::make_shared<Expr>(Mul{left, right});
}

ExprPtr sub(ExprPtr left, ExprPtr right) {
    return std::make_shared<Expr>(Sub{left, right});
}

int eval(const Expr& expr);

int eval(const Expr& expr) {
    return std::visit(overload{
        [](const Literal& l) { return l.value; },
        [](const Add& a) { return eval(*a.left) + eval(*a.right); },
        [](const Mul& m) { return eval(*m.left) * eval(*m.right); },
        [](const Sub& s) { return eval(*s.left) - eval(*s.right); }
    }, expr.data);
}

std::string to_string(const Expr& expr);

std::string to_string(const Expr& expr) {
    return std::visit(overload{
        [](const Literal& l) { return std::to_string(l.value); },
        [](const Add& a) { return "(" + to_string(*a.left) + " + " + to_string(*a.right) + ")"; },
        [](const Mul& m) { return "(" + to_string(*m.left) + " * " + to_string(*m.right) + ")"; },
        [](const Sub& s) { return "(" + to_string(*s.left) + " - " + to_string(*s.right) + ")"; }
    }, expr.data);
}

void expression_tree_demo() {
    std::cout << "=== Expression Tree ===\n";

    // (2 + 3) * (5 - 1)
    auto expr = mul(
        add(lit(2), lit(3)),
        sub(lit(5), lit(1))
    );

    std::cout << "Expression: " << to_string(*expr) << "\n";
    std::cout << "Result: " << eval(*expr) << "\n\n";
}
```

**要点**:
- 递归 ADT 用于树状结构
- 每个节点是 variant 的一种可能
- `eval` 递归求值,模式匹配处理每种节点
- `to_string` 同样递归,生成字符串表示

### 8. API 响应处理示例

```cpp
// ============================================
// 8. API Response Example
// ============================================

struct Success { std::string data; };
struct NotFound { std::string resource; };
struct ServerError { int code; std::string message; };

using ApiResponse = std::variant<Success, NotFound, ServerError>;

void handle_response(const ApiResponse& resp) {
    std::visit(overload{
        [](const Success& s) {
            std::cout << "Success: " << s.data << "\n";
        },
        [](const NotFound& nf) {
            std::cout << "404: " << nf.resource << " not found\n";
        },
        [](const ServerError& se) {
            std::cout << "Error " << se.code << ": " << se.message << "\n";
        }
    }, resp);
}

void api_response_demo() {
    std::cout << "=== API Response Handling ===\n";

    std::vector<ApiResponse> responses{
        Success{"User data retrieved"},
        NotFound{"/api/user/999"},
        ServerError{500, "Internal server error"}
    };

    for (const auto& resp : responses) {
        handle_response(resp);
    }
    std::cout << "\n";
}
```

**要点**:
- 用 variant 建模 API 不同响应类型
- 比 HTTP 状态码 + 字符串解析更类型安全
- 每种响应携带相关数据
- 模式匹配集中处理不同情况

### 运行完整示例

```cpp
// ============================================
// Main
// ============================================

int main() {
    product_types_demo();
    sum_types_demo();
    optional_demo();
    pattern_matching_demo();
    result_type_demo();
    state_machine_demo();
    expression_tree_demo();
    api_response_demo();

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
./bin/06_algebraic_types
```

**GitHub 完整源码**: [06_algebraic_types.cpp](https://github.com/wzxzhuxi/cpp-functional-programming/blob/main/examples/advanced/06_algebraic_types.cpp)

### 示例输出

```
=== Product Types ===
Alice is 25 years old, 1.65m tall
Bob scored 95

=== Sum Types (variant) ===
Value: 42
Value: 3.14
Value: hello

=== Optional ===
Found: 3
Not found: 10

=== Pattern Matching ===
Integer: 42
Double: 3.14159
String: test

=== Result Type ===
10 / 2 = 5
10 / 0 error: Division by zero
sqrt(16) = 4
sqrt(-4) error: Negative sqrt

=== State Machine ===
State: Idle
State: Running (25%)
State: Running (50%)
State: Running (75%)
State: Completed - Success!
State: Failed - Network timeout

=== Expression Tree ===
Expression: ((2 + 3) * (5 - 1))
Result: 20

=== API Response Handling ===
Success: User data retrieved
404: /api/user/999 not found
Error 500: Internal server error
```

### 关键要点总结

1. **积类型组合值**:Struct、Tuple、Pair 表示 AND 关系
2. **和类型表示选择**:Variant、Optional 表示 OR 关系
3. **Overload 模式简化访问**:优雅的模式匹配
4. **Result 类型错误处理**:显式、类型安全的错误
5. **递归 ADT 建模树**:表达式、AST 等结构
6. **状态机建模**:只允许合法状态
7. **避免非法状态**:类型系统保证正确性
8. **穷举检查**:编译器确保处理所有情况

---

## 练习题

本章提供了配套的练习题,帮助你巩固所学知识:

1. 用 variant 实现简单的 JSON 值类型
2. 实现带缓存的 optional(Lazy Maybe)
3. 用 Result 重写异常代码
4. 实现计算器的 AST 和求值器
5. 用状态机建模订单处理流程
6. 实现 Either Monad(Result 的 Functor)

**[查看练习题 →](https://github.com/wzxzhuxi/cpp-functional-programming/blob/main/exercises/06_algebraic_types_exercises.cpp)**

练习题包含:
- 详细的问题描述
- 测试用例和预期输出
- 难度分级和提示

### 参考答案

⚠️ **建议先独立完成练习再查看答案**

**[查看参考答案 →](https://github.com/wzxzhuxi/cpp-functional-programming/blob/main/exercises/solutions/06_algebraic_types_solutions.cpp)**

答案包含:
- 完整的实现代码
- 详细的解题思路
- 常见错误分析
