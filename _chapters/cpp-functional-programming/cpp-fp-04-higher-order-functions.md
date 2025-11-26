---
title: 第四章：高阶函数
book: cpp-functional-programming
order: 4
chapter_number: 4
difficulty: 进阶
summary: 什么是高阶函数、Map/Filter/Reduce、柯里化、偏函数应用、函数组合
permalink: /books/cpp-functional-programming/04-higher-order-functions/
estimated_time: "45分钟"
github_exercises: "https://github.com/wzxzhuxi/cpp-functional-programming/blob/main/exercises/04_higher_order_functions_exercises.cpp"
github_solutions: "https://github.com/wzxzhuxi/cpp-functional-programming/blob/main/exercises/solutions/04_higher_order_functions_solutions.cpp"
---

## 理论概念

### 什么是高阶函数

**高阶函数(Higher-Order Function)**：满足以下至少一个条件的函数。

1. **接受函数作为参数**
2. **返回函数作为结果**

```cpp
// 高阶函数：接受函数作为参数
int apply(int x, int (*f)(int)) {
    return f(x);
}

// 高阶函数：返回函数
auto make_adder(int n) {
    return [n](int x) { return x + n; };
}

// 既接受又返回函数
auto compose(auto f, auto g) {
    return [f, g](auto x) { return f(g(x)); };
}
```

高阶函数是函数式编程的核心工具。

### 函数作为一等公民

C++ 中函数是一等公民：可以像普通值一样使用。

#### 函数作为参数

```cpp
// 传递函数指针
int apply_twice(int x, int (*f)(int)) {
    return f(f(x));
}

int square(int x) { return x * x; }
apply_twice(3, square);  // square(square(3)) = 81

// 传递 lambda
auto result = apply_twice(3, [](int x) { return x + 1; });  // 5

// 传递 std::function
void execute(std::function<void()> action) {
    action();
}

execute([]{ std::cout << "Hello\n"; });
```

#### 函数作为返回值

```cpp
// 返回 lambda
auto make_multiplier(int factor) {
    return [factor](int x) { return x * factor; };
}

auto times3 = make_multiplier(3);
times3(10);  // 30

// 返回复杂函数
auto make_validator(int min, int max) {
    return [min, max](int x) {
        return x >= min && x <= max;
    };
}

auto is_valid_age = make_validator(0, 150);
is_valid_age(25);   // true
is_valid_age(200);  // false
```

#### 存储函数

```cpp
// 使用 std::function 存储
std::vector<std::function<int(int)>> operations;

operations.push_back([](int x) { return x * 2; });
operations.push_back([](int x) { return x + 10; });
operations.push_back([](int x) { return x * x; });

int result = 5;
for (const auto& op : operations) {
    result = op(result);
}
// result = ((5 * 2) + 10)^2 = 400
```

### Map、Filter、Reduce

三个最基础的高阶函数。

#### Map：转换

将函数应用到每个元素,返回新集合。

```cpp
template<typename T, typename Func>
auto map(const std::vector<T>& vec, Func f) {
    using R = decltype(f(std::declval<T>()));
    std::vector<R> result;
    result.reserve(vec.size());
    for (const auto& item : vec) {
        result.push_back(f(item));
    }
    return result;
}

// 使用
std::vector<int> nums = {1, 2, 3, 4, 5};
auto squares = map(nums, [](int x) { return x * x; });
// squares = {1, 4, 9, 16, 25}
```

#### Filter：过滤

保留满足条件的元素。

```cpp
template<typename T, typename Pred>
std::vector<T> filter(const std::vector<T>& vec, Pred pred) {
    std::vector<T> result;
    for (const auto& item : vec) {
        if (pred(item)) {
            result.push_back(item);
        }
    }
    return result;
}

// 使用
auto evens = filter(nums, [](int x) { return x % 2 == 0; });
// evens = {2, 4}
```

#### Reduce：归约

将集合归约为单个值。

```cpp
template<typename T, typename Acc, typename Func>
Acc reduce(const std::vector<T>& vec, Acc init, Func f) {
    Acc result = init;
    for (const auto& item : vec) {
        result = f(result, item);
    }
    return result;
}

// 使用
auto sum = reduce(nums, 0, [](int acc, int x) { return acc + x; });
// sum = 15

auto product = reduce(nums, 1, [](int acc, int x) { return acc * x; });
// product = 120
```

#### STL 等价物

C++ 标准库已经提供了这些功能：

```cpp
#include <algorithm>
#include <numeric>

// map → std::transform
std::vector<int> squares;
std::transform(nums.begin(), nums.end(),
               std::back_inserter(squares),
               [](int x) { return x * x; });

// filter → std::copy_if
std::vector<int> evens;
std::copy_if(nums.begin(), nums.end(),
             std::back_inserter(evens),
             [](int x) { return x % 2 == 0; });

// reduce → std::accumulate
int sum = std::accumulate(nums.begin(), nums.end(), 0);
int product = std::accumulate(nums.begin(), nums.end(), 1,
                               std::multiplies<>());
```

### 柯里化(Currying)

将多参数函数转换为一系列单参数函数。

```cpp
// 原始函数：多参数
int add(int a, int b, int c) {
    return a + b + c;
}

// 柯里化版本
auto curry_add(int a) {
    return [a](int b) {
        return [a, b](int c) {
            return a + b + c;
        };
    };
}

// 使用
auto step1 = curry_add(1);      // 部分应用 a=1
auto step2 = step1(2);          // 部分应用 b=2
int result = step2(3);          // 完全应用 c=3,结果 6

// 或一次性调用
int result2 = curry_add(1)(2)(3);  // 6
```

#### 通用柯里化

```cpp
// 两参数柯里化
template<typename F>
auto curry2(F f) {
    return [f](auto a) {
        return [f, a](auto b) {
            return f(a, b);
        };
    };
}

// 使用
auto multiply = [](int a, int b) { return a * b; };
auto curried_mult = curry2(multiply);

auto times5 = curried_mult(5);
times5(10);  // 50
times5(3);   // 15
```

#### 为什么柯里化有用

1. **部分应用** - 固定部分参数,创建新函数
2. **函数组合** - 更容易组合单参数函数
3. **代码复用** - 通用函数 → 特化函数

```cpp
// 通用计算价格函数
auto calc_price = curry2([](double rate, double price) {
    return price * (1 + rate);
});

// 特化：不同税率
auto with_vat = calc_price(0.2);      // 20% 税率
auto with_sales_tax = calc_price(0.1);  // 10% 税率

with_vat(100);         // 120
with_sales_tax(100);   // 110
```

### 偏函数应用(Partial Application)

固定函数的部分参数,返回新函数。

```cpp
// 使用 std::bind
auto multiply = [](int a, int b) { return a * b; };

auto double_it = std::bind(multiply, 2, std::placeholders::_1);
double_it(5);  // 10

auto add_three = [](int a, int b, int c) { return a + b + c; };
auto add_to_10 = std::bind(add_three, std::placeholders::_1, std::placeholders::_2, 10);
add_to_10(5, 3);  // 18
```

#### 手动实现偏应用

```cpp
// 简单的偏应用
template<typename Func, typename Arg>
auto partial(Func f, Arg arg) {
    return [f, arg](auto... rest) {
        return f(arg, rest...);
    };
}

// 使用
auto power = [](int base, int exp) {
    int result = 1;
    for (int i = 0; i < exp; ++i) result *= base;
    return result;
};

auto square = partial(power, 2);  // 固定 base=2
square(3);  // 2^3 = 8
square(4);  // 2^4 = 16

auto cube_of = partial([](int exp, int base) {
    return std::pow(base, exp);
}, 3);
cube_of(2);  // 2^3 = 8
cube_of(5);  // 5^3 = 125
```

### 函数组合

组合多个函数形成新函数。

```cpp
// compose: (f ∘ g)(x) = f(g(x))
template<typename F, typename G>
auto compose(F f, G g) {
    return [f, g](auto x) { return f(g(x)); };
}

// 使用
auto add10 = [](int x) { return x + 10; };
auto double_it = [](int x) { return x * 2; };

auto double_then_add10 = compose(add10, double_it);
double_then_add10(5);  // (5 * 2) + 10 = 20

// 多层组合
auto square = [](int x) { return x * x; };
auto complex = compose(compose(add10, double_it), square);
complex(3);  // ((3^2) * 2) + 10 = 28
```

#### Pipe：反向组合

```cpp
// pipe: x |> f |> g = g(f(x))
template<typename F, typename G>
auto pipe(F f, G g) {
    return [f, g](auto x) { return g(f(x)); };
}

// 更自然的流式调用
auto process = pipe(
    pipe(square, double_it),
    add10
);
process(3);  // 28,同上

// C++20 ranges 提供了 | 语法
// nums | std::views::transform(f) | std::views::filter(pred)
```

### 实用高阶函数

#### ForEach

```cpp
template<typename T, typename Action>
void for_each(const std::vector<T>& vec, Action action) {
    for (const auto& item : vec) {
        action(item);
    }
}

for_each(nums, [](int x) {
    std::cout << x << " ";
});
```

#### Any / All

```cpp
template<typename T, typename Pred>
bool any(const std::vector<T>& vec, Pred pred) {
    for (const auto& item : vec) {
        if (pred(item)) return true;
    }
    return false;
}

template<typename T, typename Pred>
bool all(const std::vector<T>& vec, Pred pred) {
    for (const auto& item : vec) {
        if (!pred(item)) return false;
    }
    return true;
}

any(nums, [](int x) { return x > 10; });  // false
all(nums, [](int x) { return x > 0; });   // true
```

#### TakeWhile / DropWhile

```cpp
template<typename T, typename Pred>
std::vector<T> take_while(const std::vector<T>& vec, Pred pred) {
    std::vector<T> result;
    for (const auto& item : vec) {
        if (!pred(item)) break;
        result.push_back(item);
    }
    return result;
}

std::vector<int> nums = {1, 2, 3, 4, 5, 6};
auto taken = take_while(nums, [](int x) { return x < 4; });
// taken = {1, 2, 3}
```

### 实践建议

1. **优先用 STL** - `std::transform`、`std::accumulate` 等已优化
2. **lambda 简洁** - 短小的 lambda 比命名函数更清晰
3. **类型推导** - 用 `auto` 和模板,避免写复杂类型
4. **避免过度抽象** - 高阶函数很强大,但别滥用
5. **性能注意** - `std::function` 有开销,性能敏感用模板

### 关键要点总结

1. **高阶函数 = 抽象工具** - 操作函数的函数
2. **函数是一等公民** - 可以传递、返回、存储
3. **Map/Filter/Reduce** - 最基础的三个模式
4. **柯里化和偏应用** - 固定部分参数,创建特化函数
5. **函数组合** - 小函数组合成大函数
6. **STL 已提供** - 不需要重复造轮子

---

## 代码示例

### 代码示例概览

本节包含高阶函数的完整可运行示例,涵盖：

1. **一等公民函数** - 传递、返回、存储函数
2. **Map/Filter/Reduce** - 三大基础高阶函数
3. **柯里化** - 多参数转单参数
4. **偏函数应用** - 固定部分参数
5. **函数组合** - 组合小函数成大函数
6. **实用工具函数** - Any/All/TakeWhile等
7. **函数存储** - 使用 std::function
8. **实际应用** - 数据处理管道

### 示例 1：一等公民函数

```cpp
// 接受函数作为参数
int apply_twice(int x, int (*f)(int)) {
    return f(f(x));
}

int square_func(int x) { return x * x; }

// 返回函数
auto make_multiplier(int factor) {
    return [factor](int x) { return x * factor; };
}

auto make_validator(int min, int max) {
    return [min, max](int x) {
        return x >= min && x <= max;
    };
}

void first_class_functions_demo() {
    // 函数作为参数
    std::cout << "apply_twice(3, square) = " << apply_twice(3, square_func) << "\n";
    std::cout << "apply_twice(3, +1) = " << apply_twice(3, [](int x) { return x + 1; }) << "\n";

    // 函数作为返回值
    auto times5 = make_multiplier(5);
    std::cout << "times5(10) = " << times5(10) << "\n";

    auto is_valid_age = make_validator(0, 150);
    std::cout << "is_valid_age(25) = " << is_valid_age(25) << "\n";
    std::cout << "is_valid_age(200) = " << is_valid_age(200) << "\n";
}
```

**要点**:
- 函数可以像普通值一样传递和返回
- Lambda 表达式使函数操作更简洁
- 高阶函数提供强大的抽象能力

### 示例 2：Map/Filter/Reduce

```cpp
template<typename T, typename Func>
auto map(const std::vector<T>& vec, Func f) {
    using R = decltype(f(std::declval<T>()));
    std::vector<R> result;
    result.reserve(vec.size());
    for (const auto& item : vec) {
        result.push_back(f(item));
    }
    return result;
}

template<typename T, typename Pred>
std::vector<T> filter(const std::vector<T>& vec, Pred pred) {
    std::vector<T> result;
    for (const auto& item : vec) {
        if (pred(item)) {
            result.push_back(item);
        }
    }
    return result;
}

template<typename T, typename Acc, typename Func>
Acc reduce(const std::vector<T>& vec, Acc init, Func f) {
    Acc result = init;
    for (const auto& item : vec) {
        result = f(result, item);
    }
    return result;
}

void map_filter_reduce_demo() {
    std::vector<int> nums = {1, 2, 3, 4, 5};

    // Map
    auto squares = map(nums, [](int x) { return x * x; });
    std::cout << "Squares: ";
    for (int x : squares) std::cout << x << " ";
    std::cout << "\n";

    // Filter
    auto evens = filter(nums, [](int x) { return x % 2 == 0; });
    std::cout << "Evens: ";
    for (int x : evens) std::cout << x << " ";
    std::cout << "\n";

    // Reduce
    auto sum = reduce(nums, 0, [](int acc, int x) { return acc + x; });
    auto product = reduce(nums, 1, [](int acc, int x) { return acc * x; });
    std::cout << "Sum: " << sum << "\n";
    std::cout << "Product: " << product << "\n";

    // 链式调用
    auto result = reduce(
        filter(
            map(nums, [](int x) { return x * 2; }),
            [](int x) { return x > 5; }
        ),
        0,
        [](int acc, int x) { return acc + x; }
    );
    std::cout << "Chain result: " << result << " (double, filter >5, sum)\n";
}
```

**要点**:
- Map 转换每个元素
- Filter 筛选满足条件的元素
- Reduce 将集合归约为单个值
- 可以链式组合使用

### 示例 3：柯里化

```cpp
// 手动柯里化
auto curry_add(int a) {
    return [a](int b) {
        return [a, b](int c) {
            return a + b + c;
        };
    };
}

// 通用两参数柯里化
template<typename F>
auto curry2(F f) {
    return [f](auto a) {
        return [f, a](auto b) {
            return f(a, b);
        };
    };
}

void currying_demo() {
    // 手动柯里化
    auto step1 = curry_add(1);
    auto step2 = step1(2);
    auto result = step2(3);
    std::cout << "curry_add(1)(2)(3) = " << result << "\n";

    // 通用柯里化
    auto multiply = [](int a, int b) { return a * b; };
    auto curried_mult = curry2(multiply);

    auto times10 = curried_mult(10);
    std::cout << "times10(5) = " << times10(5) << "\n";
    std::cout << "times10(7) = " << times10(7) << "\n";

    // 实用案例：价格计算
    auto calc_price = curry2([](double rate, double price) {
        return price * (1 + rate);
    });

    auto with_vat = calc_price(0.2);
    auto with_sales_tax = calc_price(0.1);

    std::cout << "with_vat(100) = " << with_vat(100) << "\n";
    std::cout << "with_sales_tax(100) = " << with_sales_tax(100) << "\n";
}
```

**要点**:
- 柯里化将多参数函数变为嵌套的单参数函数
- 便于部分应用和函数复用
- 实际应用中可创建特化版本

### 示例 4：偏函数应用

```cpp
template<typename Func, typename Arg>
auto partial(Func f, Arg arg) {
    return [f, arg](auto... rest) {
        return f(arg, rest...);
    };
}

void partial_application_demo() {
    // 使用 std::bind
    auto multiply = [](int a, int b) { return a * b; };
    auto double_it = std::bind(multiply, 2, std::placeholders::_1);
    std::cout << "double_it(5) = " << double_it(5) << "\n";

    // 手动 partial
    auto power = [](int base, int exp) {
        int result = 1;
        for (int i = 0; i < exp; ++i) result *= base;
        return result;
    };

    auto powers_of_2 = partial(power, 2);
    std::cout << "powers_of_2(3) = " << powers_of_2(3) << " (2^3)\n";
    std::cout << "powers_of_2(5) = " << powers_of_2(5) << " (2^5)\n";

    auto cube_of = partial([](int exp, int base) {
        return static_cast<int>(std::pow(base, exp));
    }, 3);
    std::cout << "cube_of(2) = " << cube_of(2) << " (2^3)\n";
    std::cout << "cube_of(4) = " << cube_of(4) << " (4^3)\n";
}
```

**要点**:
- 偏应用固定部分参数,返回新函数
- 可用 std::bind 或手动实现
- 适合创建专用函数

### 示例 5：函数组合

```cpp
template<typename F, typename G>
auto compose(F f, G g) {
    return [f, g](auto x) { return f(g(x)); };
}

template<typename F, typename G>
auto pipe(F f, G g) {
    return [f, g](auto x) { return g(f(x)); };
}

void composition_demo() {
    auto add10 = [](int x) { return x + 10; };
    auto double_it = [](int x) { return x * 2; };
    auto square = [](int x) { return x * x; };

    // compose: f(g(x))
    auto double_then_add10 = compose(add10, double_it);
    std::cout << "double_then_add10(5) = " << double_then_add10(5)
              << " ((5*2)+10)\n";

    // pipe: g(f(x))
    auto add10_then_double = pipe(add10, double_it);
    std::cout << "add10_then_double(5) = " << add10_then_double(5)
              << " ((5+10)*2)\n";

    // 多层组合
    auto complex = compose(add10, compose(double_it, square));
    std::cout << "complex(3) = " << complex(3)
              << " (((3^2)*2)+10)\n";
}
```

**要点**:
- Compose 从右到左组合函数: f(g(x))
- Pipe 从左到右组合函数: g(f(x))
- 可以组合多个小函数构建复杂逻辑

### 示例 6：实用高阶函数

```cpp
template<typename T, typename Pred>
bool any(const std::vector<T>& vec, Pred pred) {
    for (const auto& item : vec) {
        if (pred(item)) return true;
    }
    return false;
}

template<typename T, typename Pred>
bool all(const std::vector<T>& vec, Pred pred) {
    for (const auto& item : vec) {
        if (!pred(item)) return false;
    }
    return true;
}

template<typename T, typename Pred>
std::vector<T> take_while(const std::vector<T>& vec, Pred pred) {
    std::vector<T> result;
    for (const auto& item : vec) {
        if (!pred(item)) break;
        result.push_back(item);
    }
    return result;
}

void utility_functions_demo() {
    std::vector<int> nums = {1, 2, 3, 4, 5, 6};

    std::cout << "any > 5: " << any(nums, [](int x) { return x > 5; }) << "\n";
    std::cout << "all > 0: " << all(nums, [](int x) { return x > 0; }) << "\n";

    auto taken = take_while(nums, [](int x) { return x < 4; });
    std::cout << "take_while < 4: ";
    for (int x : taken) std::cout << x << " ";
    std::cout << "\n";
}
```

**要点**:
- Any 检查是否有元素满足条件
- All 检查是否所有元素满足条件
- TakeWhile 取满足条件的前缀元素

### 示例 7：存储和执行函数

```cpp
void function_storage_demo() {
    std::vector<std::function<int(int)>> operations;

    operations.push_back([](int x) { return x * 2; });
    operations.push_back([](int x) { return x + 10; });
    operations.push_back([](int x) { return x * x; });

    int value = 5;
    std::cout << "Starting value: " << value << "\n";

    for (const auto& op : operations) {
        value = op(value);
        std::cout << "After operation: " << value << "\n";
    }

    std::cout << "Final: " << value << " (((5*2)+10)^2 = 400)\n";
}
```

**要点**:
- std::function 可以存储不同类型的可调用对象
- 适合需要动态选择和执行函数的场景
- 有一定性能开销,性能敏感场景用模板

### 示例 8：实际应用 - 数据处理管道

```cpp
struct Person {
    std::string name;
    int age;
    double salary;
};

void pipeline_demo() {
    std::vector<Person> people = {
        {"Alice", 30, 75000},
        {"Bob", 25, 60000},
        {"Charlie", 35, 90000},
        {"Diana", 28, 70000},
        {"Eve", 32, 85000}
    };

    // 管道：过滤年龄 > 28 -> 提取薪水 -> 求和
    auto salaries = map(
        filter(people, [](const Person& p) { return p.age > 28; }),
        [](const Person& p) { return p.salary; }
    );

    auto total = reduce(salaries, 0.0, [](double acc, double s) { return acc + s; });

    std::cout << "Total salary (age > 28): " << total << "\n";

    // 另一个管道：薪水加 10% -> 过滤 > 80000
    auto raised_salaries = map(people, [](const Person& p) {
        return Person{p.name, p.age, p.salary * 1.1};
    });

    auto high_earners = filter(raised_salaries, [](const Person& p) {
        return p.salary > 80000;
    });

    std::cout << "High earners after 10% raise:\n";
    for (const auto& p : high_earners) {
        std::cout << "  " << p.name << ": " << p.salary << "\n";
    }
}
```

**要点**:
- 高阶函数可构建清晰的数据处理管道
- 声明式风格更易理解和维护
- 链式调用使数据流向清晰

### 运行完整示例

下载并运行：

```bash
git clone https://github.com/wzxzhuxi/cpp-functional-programming.git
cd cpp-functional-programming
mkdir build && cd build
cmake ..
make
./bin/04_higher_order_functions
```

**完整源码**: [04_higher_order_functions.cpp](https://github.com/wzxzhuxi/cpp-functional-programming/blob/main/examples/intermediate/04_higher_order_functions.cpp)

### 关键要点总结

1. **高阶函数 = 元编程** - 用函数操作函数
2. **Map/Filter/Reduce** - 数据处理的基础模式
3. **柯里化和偏应用** - 创建特化函数,提高复用性
4. **函数组合** - 小函数拼接成大函数
5. **STL 优先** - 标准库已提供优化实现
6. **实际应用** - 构建清晰的数据处理管道

---

## 练习与答案

本章提供了配套的练习题和参考答案,帮助你巩固所学知识。

### 练习题

在线查看练习题和测试用例：

**[查看练习题 →](https://github.com/wzxzhuxi/cpp-functional-programming/blob/main/exercises/04_higher_order_functions_exercises.cpp)**

练习包含：
- 详细的问题描述
- 测试用例和预期输出
- 难度分级和提示

### 参考答案

警告 **建议先独立完成练习再查看答案**

**[查看参考答案 →](https://github.com/wzxzhuxi/cpp-functional-programming/blob/main/exercises/solutions/04_higher_order_functions_solutions.cpp)**

答案包含：
- 完整的实现代码
- 详细的解题思路
- 常见错误分析
