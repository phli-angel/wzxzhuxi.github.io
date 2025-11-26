---
title: 第五章:函数组合
book: cpp-functional-programming
order: 5
chapter_number: 5
difficulty: 进阶
summary: Compose和Pipe、数据流转换、点自由风格、组合子
permalink: /books/cpp-functional-programming/05-functional-composition/
estimated_time: "60分钟"
github_exercises: "https://github.com/wzxzhuxi/cpp-functional-programming/tree/main/exercises"
github_solutions: "https://github.com/wzxzhuxi/cpp-functional-programming/tree/main/exercises/solutions"
---

## 说人话:函数组合是什么?

函数组合不是什么高深的技巧。**就是把多个小函数拼成一个大函数**。

就像乐高积木:
- 每个小函数做一件事(一个积木块)
- 组合起来完成复杂任务(拼成城堡)

**核心思想**:用小函数构建大系统。

### 数学定义

在数学中,函数组合定义为:
```
(f ∘ g)(x) = f(g(x))
```

在 C++ 中,我们可以实现类似的机制。

## 1. Compose 和 Pipe

### Compose(从右到左)

```cpp
template<typename F, typename G>
auto compose(F f, G g) {
    return [f, g](auto x) {
        return f(g(x));
    };
}

// 使用
auto add10 = [](int x) { return x + 10; };
auto double_it = [](int x) { return x * 2; };

auto f = compose(add10, double_it);
// f(5) = add10(double_it(5)) = add10(10) = 20
```

**执行顺序**:从右到左(符合数学习惯)

### Pipe(从左到右)

```cpp
template<typename F, typename G>
auto pipe(F f, G g) {
    return [f, g](auto x) {
        return g(f(x));
    };
}

// 使用
auto f = pipe(double_it, add10);
// f(5) = add10(double_it(5)) = 20
```

**执行顺序**:从左到右(符合数据流动方向)

### 多函数组合

```cpp
// 可变参数模板实现
template<typename F>
auto compose_many(F f) {
    return f;
}

template<typename F, typename... Rest>
auto compose_many(F f, Rest... rest) {
    return [f, rest...](auto x) {
        return f(compose_many(rest...)(x));
    };
}

// 使用
auto f = compose_many(add10, double_it, square);
// f(3) = add10(double_it(square(3)))
//      = add10(double_it(9))
//      = add10(18)
//      = 28
```

## 2. 数据流转换

### 管道思维

将数据处理看作流水线:

```cpp
// 不使用组合
auto result = step3(step2(step1(data)));  // 难读

// 使用 pipe
auto pipeline = pipe(step1, step2, step3);
auto result = pipeline(data);  // 清晰
```

### 实际案例:文本处理

```cpp
auto trim = [](std::string s) {
    // 去除首尾空格
    s.erase(0, s.find_first_not_of(" \t\n\r"));
    s.erase(s.find_last_not_of(" \t\n\r") + 1);
    return s;
};

auto to_upper = [](std::string s) {
    std::transform(s.begin(), s.end(), s.begin(), ::toupper);
    return s;
};

auto remove_spaces = [](std::string s) {
    s.erase(std::remove(s.begin(), s.end(), ' '), s.end());
    return s;
};

// 组合成处理管道
auto normalize = pipe(trim, to_upper, remove_spaces);

std::string input = "  Hello World  ";
auto result = normalize(input);  // "HELLOWORLD"
```

## 3. 点自由风格(Point-Free Style)

点自由风格:不显式提及函数参数。

### 对比

```cpp
// 显式参数风格(point-ful)
auto process = [](int x) {
    return add10(double_it(x));
};

// 点自由风格(point-free)
auto process = compose(add10, double_it);
```

### 优势

1. **更简洁**:不需要命名临时变量
2. **更抽象**:关注操作本身,而非数据
3. **更易组合**:函数成为"乐高积木"

### 何时使用

✅ **适合使用点自由风格**:
- 简单的函数组合
- 逻辑清晰的管道
- 可读性不受影响

❌ **避免使用点自由风格**:
- 过于复杂的组合(难以理解)
- 需要中间调试
- 多个参数需要重排

## 4. 函数组合的实际应用

### 4.1 数据验证管道

```cpp
struct User {
    std::string name;
    std::string email;
    int age;
};

// 各种验证函数
auto validate_name = [](const User& u) -> std::optional<User> {
    if (u.name.empty()) return std::nullopt;
    return u;
};

auto validate_email = [](const User& u) -> std::optional<User> {
    if (u.email.find('@') == std::string::npos) return std::nullopt;
    return u;
};

auto validate_age = [](const User& u) -> std::optional<User> {
    if (u.age < 0 || u.age > 150) return std::nullopt;
    return u;
};

// 组合验证器(需要处理 optional)
auto validate_user = [](const User& u) {
    return validate_name(u)
        .and_then(validate_email)
        .and_then(validate_age);
};
```

### 4.2 数据转换链

```cpp
// 产品价格计算
auto apply_discount = [](double rate) {
    return [rate](double price) {
        return price * (1.0 - rate);
    };
};

auto apply_tax = [](double rate) {
    return [rate](double price) {
        return price * (1.0 + rate);
    };
};

auto round_to_cents = [](double price) {
    return std::round(price * 100) / 100.0;
};

// 价格计算管道
auto calculate_final_price = pipe(
    apply_discount(0.1),   // 10% 折扣
    apply_tax(0.2),        // 20% 税
    round_to_cents         // 四舍五入到分
);

double original_price = 100.0;
double final_price = calculate_final_price(original_price);
// 100 -> 90 -> 108 -> 108.00
```

### 4.3 函数式错误处理

```cpp
template<typename T>
using Result = std::variant<T, std::string>;

// 安全除法
auto safe_divide = [](double a, double b) -> Result<double> {
    if (b == 0) return std::string("Division by zero");
    return a / b;
};

// 安全平方根
auto safe_sqrt = [](double x) -> Result<double> {
    if (x < 0) return std::string("Negative sqrt");
    return std::sqrt(x);
};

// 组合(需要 monadic bind)
auto safe_operation = [](double a, double b) {
    auto div_result = safe_divide(a, b);
    if (std::holds_alternative<std::string>(div_result)) {
        return div_result;
    }
    return safe_sqrt(std::get<double>(div_result));
};
```

## 5. 组合子(Combinators)

组合子是操作函数的高阶函数。

### 常用组合子

```cpp
// K 组合子:常量函数
template<typename T>
auto K(T value) {
    return [value](auto...) { return value; };
}

// I 组合子:恒等函数
auto I = [](auto x) { return x; };

// S 组合子:应用组合
template<typename F, typename G>
auto S(F f, G g) {
    return [f, g](auto x) {
        return f(x)(g(x));
    };
}

// B 组合子:就是 compose
template<typename F, typename G>
auto B(F f, G g) {
    return compose(f, g);
}
```

### 实用组合子

```cpp
// flip:翻转二元函数的参数
template<typename F>
auto flip(F f) {
    return [f](auto a, auto b) {
        return f(b, a);
    };
}

// on:在应用函数前先转换
template<typename F, typename G>
auto on(F f, G g) {
    return [f, g](auto a, auto b) {
        return f(g(a), g(b));
    };
}

// 使用示例
auto subtract = [](int a, int b) { return a - b; };
auto flipped_sub = flip(subtract);
// flipped_sub(5, 3) = 3 - 5 = -2

auto compare_length = on(
    [](size_t a, size_t b) { return a < b; },
    [](const std::string& s) { return s.length(); }
);
// compare_length("hi", "hello") = true
```

## 6. 函数组合的陷阱

### 过度组合

❌ **糟糕**:
```cpp
auto f = compose(a, compose(b, compose(c, compose(d, compose(e, f)))));
// 难以阅读和调试
```

✅ **改进**:
```cpp
auto step1 = compose(b, a);
auto step2 = compose(d, c);
auto step3 = compose(f, e);
auto final = compose(step3, compose(step2, step1));
// 分步骤,易于理解
```

### 类型不匹配

```cpp
auto add10 = [](int x) { return x + 10; };
auto to_string = [](int x) { return std::to_string(x); };

// 错误:add10 接受 int,但 to_string 返回 string
// auto bad = compose(add10, to_string);  // 编译错误
```

### 性能考虑

每层组合都会创建新的 lambda,可能影响性能。对性能敏感的代码,考虑:
1. 手动内联
2. 使用编译器优化(`-O3`)
3. 避免过深的组合链

---

## 最佳实践

1. **保持简单**:每个函数做一件事
2. **命名清晰**:组合的函数应有描述性名称
3. **类型安全**:利用 C++ 类型系统捕获错误
4. **可测试**:分别测试每个组件,再测试组合
5. **文档化**:复杂组合需要注释说明数据流

## 为什么函数组合有用?

### 1. 可读性
管道式代码比嵌套调用更清晰。

### 2. 可复用性
小函数可以在不同场景重组。

### 3. 可测试性
每个小函数独立测试,组合后整体测试。

### 4. 可维护性
修改一个小函数,所有使用它的组合自动更新。

## 总结

函数组合是构建复杂系统的基础:
- **compose/pipe**:基本组合工具
- **点自由风格**:更抽象的代码
- **管道思维**:数据流转换
- **组合子**:操作函数的函数

掌握函数组合,就能用小函数构建大系统。

---

## 完整代码示例

本节包含完整可运行示例,涵盖所有核心概念:

### 基础函数定义

```cpp
#include <algorithm>
#include <cctype>
#include <cmath>
#include <functional>
#include <iostream>
#include <optional>
#include <string>
#include <variant>
#include <vector>

// ============================================
// 1. 基础 Compose 和 Pipe
// ============================================

template<typename F, typename G>
auto compose(F f, G g) {
    return [f, g](auto x) { return f(g(x)); };
}

template<typename F, typename G>
auto pipe(F f, G g) {
    return [f, g](auto x) { return g(f(x)); };
}
```

### 1. Compose 和 Pipe 示例

```cpp
void compose_pipe_demo() {
    std::cout << "=== Compose and Pipe ===\n";

    auto add10 = [](int x) { return x + 10; };
    auto double_it = [](int x) { return x * 2; };
    auto square = [](int x) { return x * x; };

    // Compose: 从右到左
    auto f1 = compose(add10, double_it);
    std::cout << "compose(+10, *2)(5) = " << f1(5)
              << " (先 *2=10, 再 +10=20)\n";

    // Pipe: 从左到右
    auto f2 = pipe(double_it, add10);
    std::cout << "pipe(*2, +10)(5) = " << f2(5)
              << " (先 *2=10, 再 +10=20)\n";

    // 多层组合
    auto f3 = compose(add10, compose(double_it, square));
    std::cout << "compose(+10, compose(*2, ^2))(3) = " << f3(3)
              << " (3 -> 9 -> 18 -> 28)\n\n";
}
```

**要点**:
- Compose 从右到左执行,符合数学习惯 f(g(x))
- Pipe 从左到右执行,符合数据流方向
- 可以嵌套组合,但要注意可读性
- Compose 和 Pipe 只是执行顺序相反

### 2. 可变参数 Compose 示例

```cpp
// ============================================
// 2. 可变参数 Compose
// ============================================

template<typename F>
auto compose_many(F f) {
    return f;
}

template<typename F, typename... Rest>
auto compose_many(F f, Rest... rest) {
    return [f, rest...](auto x) {
        return f(compose_many(rest...)(x));
    };
}

template<typename F>
auto pipe_many(F f) {
    return f;
}

template<typename F, typename... Rest>
auto pipe_many(F f, Rest... rest) {
    return [f, rest...](auto x) {
        return pipe_many(rest...)(f(x));
    };
}

void variadic_compose_demo() {
    std::cout << "=== Variadic Compose ===\n";

    auto add5 = [](int x) { return x + 5; };
    auto mul3 = [](int x) { return x * 3; };
    auto sub2 = [](int x) { return x - 2; };

    auto f = compose_many(add5, mul3, sub2);
    std::cout << "compose_many(+5, *3, -2)(10) = " << f(10)
              << " (10 -> 8 -> 24 -> 29)\n";

    auto g = pipe_many(sub2, mul3, add5);
    std::cout << "pipe_many(-2, *3, +5)(10) = " << g(10)
              << " (10 -> 8 -> 24 -> 29)\n\n";
}
```

**要点**:
- 可变参数模板支持任意数量函数组合
- Compose_many 递归展开,从右到左执行
- Pipe_many 递归展开,从左到右执行
- 模板递归在编译期完成,运行时无开销

### 3. 文本处理管道示例

```cpp
// ============================================
// 3. 文本处理管道
// ============================================

auto trim = [](std::string s) {
    if (s.empty()) return s;
    size_t start = s.find_first_not_of(" \t\n\r");
    if (start == std::string::npos) return std::string();
    size_t end = s.find_last_not_of(" \t\n\r");
    return s.substr(start, end - start + 1);
};

auto to_upper = [](std::string s) {
    std::transform(s.begin(), s.end(), s.begin(),
                   [](unsigned char c) { return std::toupper(c); });
    return s;
};

auto to_lower = [](std::string s) {
    std::transform(s.begin(), s.end(), s.begin(),
                   [](unsigned char c) { return std::tolower(c); });
    return s;
};

auto remove_spaces = [](std::string s) {
    s.erase(std::remove(s.begin(), s.end(), ' '), s.end());
    return s;
};

void text_pipeline_demo() {
    std::cout << "=== Text Processing Pipeline ===\n";

    std::string input = "  Hello World  ";

    // 单独步骤
    std::cout << "Original: \"" << input << "\"\n";
    std::cout << "Trimmed: \"" << trim(input) << "\"\n";
    std::cout << "Upper: \"" << to_upper(trim(input)) << "\"\n";

    // 使用管道
    auto normalize = pipe_many(trim, to_upper, remove_spaces);
    std::cout << "Normalized: \"" << normalize(input) << "\"\n";

    // 不同的管道
    auto to_slug = pipe_many(trim, to_lower, remove_spaces);
    std::cout << "Slug: \"" << to_slug("  My Blog Post  ") << "\"\n\n";
}
```

**要点**:
- 文本处理是函数组合的经典应用
- 每个函数独立,可单独测试
- 管道清晰表达数据流转换
- 可灵活组合不同函数形成新管道

### 4. 点自由风格示例

```cpp
// ============================================
// 4. 点自由风格 (Point-Free)
// ============================================

void point_free_demo() {
    std::cout << "=== Point-Free Style ===\n";

    auto increment = [](int x) { return x + 1; };
    auto double_it = [](int x) { return x * 2; };

    // 显式参数 (point-ful)
    auto process_pointful = [=](int x) {
        return double_it(increment(x));
    };

    // 点自由 (point-free)
    auto process_pointfree = compose(double_it, increment);

    std::cout << "Point-ful: " << process_pointful(5) << "\n";
    std::cout << "Point-free: " << process_pointfree(5) << "\n";

    // 更复杂的例子
    std::vector<int> nums = {1, 2, 3, 4, 5};

    // 点自由风格的 map
    auto transform_pointfree = [](auto f) {
        return [f](const auto& vec) {
            std::vector<std::decay_t<decltype(vec[0])>> result;
            result.reserve(vec.size());
            std::transform(vec.begin(), vec.end(),
                         std::back_inserter(result), f);
            return result;
        };
    };

    auto doubled = transform_pointfree(double_it)(nums);
    std::cout << "Doubled: ";
    for (int x : doubled) std::cout << x << " ";
    std::cout << "\n\n";
}
```

**要点**:
- 点自由风格不提及参数,更抽象
- Point-ful 明确显示数据流,易于理解
- Point-free 更简洁,但可能牺牲可读性
- 适度使用,不要过度抽象

### 5. 组合子示例

```cpp
// ============================================
// 5. 组合子 (Combinators)
// ============================================

// K 组合子:常量函数
template<typename T>
auto K(T value) {
    return [value](auto...) { return value; };
}

// I 组合子:恒等函数
auto I = [](auto x) { return x; };

// flip:翻转参数
template<typename F>
auto flip(F f) {
    return [f](auto a, auto b) { return f(b, a); };
}

// on:在应用前转换
template<typename F, typename G>
auto on(F f, G g) {
    return [f, g](auto a, auto b) {
        return f(g(a), g(b));
    };
}

void combinators_demo() {
    std::cout << "=== Combinators ===\n";

    // K 组合子
    auto always_42 = K(42);
    std::cout << "K(42)(1, 2, 3) = " << always_42(1, 2, 3) << "\n";

    // I 组合子
    std::cout << "I(100) = " << I(100) << "\n";

    // flip
    auto subtract = [](int a, int b) { return a - b; };
    auto reversed_sub = flip(subtract);
    std::cout << "subtract(10, 3) = " << subtract(10, 3) << "\n";
    std::cout << "flip(subtract)(10, 3) = " << reversed_sub(10, 3) << "\n";

    // on
    auto compare_length = on(
        [](size_t a, size_t b) { return a < b; },
        [](const std::string& s) { return s.length(); }
    );
    std::cout << "compare_length(\"hi\", \"hello\") = "
              << std::boolalpha << compare_length("hi", "hello") << "\n\n";
}
```

**要点**:
- K 组合子返回常量,忽略所有参数
- I 组合子是恒等函数,返回输入
- Flip 翻转二元函数参数顺序
- On 在应用前先转换参数

### 6. 价格计算管道示例

```cpp
// ============================================
// 6. 实际应用:价格计算管道
// ============================================

auto apply_discount = [](double rate) {
    return [rate](double price) {
        return price * (1.0 - rate);
    };
};

auto apply_tax = [](double rate) {
    return [rate](double price) {
        return price * (1.0 + rate);
    };
};

auto round_to_cents = [](double price) {
    return std::round(price * 100.0) / 100.0;
};

void price_pipeline_demo() {
    std::cout << "=== Price Calculation Pipeline ===\n";

    double original = 100.0;

    // 逐步计算
    double after_discount = apply_discount(0.1)(original);
    double after_tax = apply_tax(0.2)(after_discount);
    double final = round_to_cents(after_tax);

    std::cout << "Original: $" << original << "\n";
    std::cout << "After 10% discount: $" << after_discount << "\n";
    std::cout << "After 20% tax: $" << after_tax << "\n";
    std::cout << "Rounded: $" << final << "\n";

    // 使用管道
    auto calculate_price = pipe_many(
        apply_discount(0.1),
        apply_tax(0.2),
        round_to_cents
    );

    std::cout << "\nUsing pipeline: $" << calculate_price(100.0) << "\n\n";
}
```

**要点**:
- 价格计算是业务逻辑的常见场景
- 高阶函数返回函数,支持参数化
- 管道清晰表达计算步骤
- 易于调整计算规则(如修改税率)

### 7. Optional 链式组合示例

```cpp
// ============================================
// 7. Optional 链式组合
// ============================================

template<typename T, typename F>
auto fmap(F f) {
    return [f](const std::optional<T>& opt) -> std::optional<std::decay_t<decltype(f(std::declval<T>()))>> {
        if (!opt) return std::nullopt;
        return f(*opt);
    };
}

void optional_composition_demo() {
    std::cout << "=== Optional Composition ===\n";

    auto safe_div = [](int a, int b) -> std::optional<int> {
        if (b == 0) return std::nullopt;
        return a / b;
    };

    auto safe_sqrt = [](int x) -> std::optional<double> {
        if (x < 0) return std::nullopt;
        return std::sqrt(x);
    };

    // 手动链式
    auto result1 = safe_div(100, 5);
    if (result1) {
        auto result2 = safe_sqrt(*result1);
        if (result2) {
            std::cout << "Manual chain: " << *result2 << "\n";
        }
    }

    // 使用 and_then (C++23) 或手动实现
    auto safe_operation = [&](int a, int b) -> std::optional<double> {
        auto div_result = safe_div(a, b);
        if (!div_result) return std::nullopt;
        return safe_sqrt(*div_result);
    };

    auto r1 = safe_operation(100, 5);
    if (r1) std::cout << "Composed: " << *r1 << "\n";

    auto r2 = safe_operation(100, 0);
    std::cout << "Failed (div by 0): "
              << (r2 ? std::to_string(*r2) : "nullopt") << "\n\n";
}
```

**要点**:
- Optional 组合避免嵌套 if 检查
- 失败时短路,跳过后续计算
- C++23 的 and_then 简化链式调用
- 适合可能失败的操作链

### 8. 函数容器示例

```cpp
// ============================================
// 8. 函数容器
// ============================================

void function_container_demo() {
    std::cout << "=== Function Container ===\n";

    std::vector<std::function<int(int)>> transformations;

    transformations.push_back([](int x) { return x + 10; });
    transformations.push_back([](int x) { return x * 2; });
    transformations.push_back([](int x) { return x - 5; });

    // 应用所有转换
    int value = 5;
    std::cout << "Initial: " << value << "\n";

    for (const auto& f : transformations) {
        value = f(value);
        std::cout << "After transformation: " << value << "\n";
    }

    // 组合所有函数
    auto combined = [&](int x) {
        for (const auto& f : transformations) {
            x = f(x);
        }
        return x;
    };

    std::cout << "Combined(5) = " << combined(5) << "\n\n";
}
```

**要点**:
- 函数可以存储在容器中
- 动态组合多个函数
- 适合运行时决定的转换链
- 可以从配置文件加载函数链

### 运行完整示例

```cpp
// ============================================
// Main
// ============================================

int main() {
    compose_pipe_demo();
    variadic_compose_demo();
    text_pipeline_demo();
    point_free_demo();
    combinators_demo();
    price_pipeline_demo();
    optional_composition_demo();
    function_container_demo();

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
./bin/05_functional_composition
```

**GitHub 完整源码**: [05_functional_composition.cpp](https://github.com/wzxzhuxi/cpp-functional-programming/blob/main/examples/intermediate/05_functional_composition.cpp)

### 示例输出

```
=== Compose and Pipe ===
compose(+10, *2)(5) = 20 (先 *2=10, 再 +10=20)
pipe(*2, +10)(5) = 20 (先 *2=10, 再 +10=20)
compose(+10, compose(*2, ^2))(3) = 28 (3 -> 9 -> 18 -> 28)

=== Variadic Compose ===
compose_many(+5, *3, -2)(10) = 29 (10 -> 8 -> 24 -> 29)
pipe_many(-2, *3, +5)(10) = 29 (10 -> 8 -> 24 -> 29)

=== Text Processing Pipeline ===
Original: "  Hello World  "
Trimmed: "Hello World"
Upper: "HELLO WORLD"
Normalized: "HELLOWORLD"
Slug: "myblogpost"

=== Point-Free Style ===
Point-ful: 12
Point-free: 12
Doubled: 2 4 6 8 10

=== Combinators ===
K(42)(1, 2, 3) = 42
I(100) = 100
subtract(10, 3) = 7
flip(subtract)(10, 3) = -7
compare_length("hi", "hello") = true

=== Price Calculation Pipeline ===
Original: $100
After 10% discount: $90
After 20% tax: $108
Rounded: $108

Using pipeline: $108

=== Optional Composition ===
Manual chain: 4.47214
Composed: 4.47214
Failed (div by 0): nullopt

=== Function Container ===
Initial: 5
After transformation: 15
After transformation: 30
After transformation: 25
Combined(5) = 25
```

### 关键要点总结

1. **Compose vs Pipe**:Compose 从右到左,Pipe 从左到右
2. **可变参数组合**:支持任意数量函数,编译期展开
3. **文本处理管道**:清晰表达数据转换流程
4. **点自由风格**:更抽象,但不要过度使用
5. **组合子**:K/I/flip/on 等高阶函数工具
6. **实际应用**:价格计算、验证链、错误处理
7. **Optional 组合**:避免嵌套检查,链式调用
8. **动态组合**:函数容器支持运行时决定

---

## 练习题

本章提供了配套的练习题,帮助你巩固所学知识:

1. 实现 compose_all 和 pipe_all,支持任意数量函数
2. 用 pipe 实现字符串处理工具链
3. 实现 curry 和 uncurry 组合子
4. 用函数组合重写嵌套的 if-else
5. 实现 Result 类型的 bind 组合
6. 用组合子实现计算器表达式求值

**[查看练习题 →](https://github.com/wzxzhuxi/cpp-functional-programming/blob/main/exercises/05_functional_composition_exercises.cpp)**

练习题包含:
- 详细的问题描述
- 测试用例和预期输出
- 难度分级和提示

### 参考答案

⚠️ **建议先独立完成练习再查看答案**

**[查看参考答案 →](https://github.com/wzxzhuxi/cpp-functional-programming/blob/main/exercises/solutions/05_functional_composition_solutions.cpp)**

答案包含:
- 完整的实现代码
- 详细的解题思路
- 常见错误分析
