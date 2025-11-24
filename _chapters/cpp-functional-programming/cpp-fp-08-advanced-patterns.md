---
title: 第八章:高级函数式模式
book: cpp-functional-programming
order: 8
chapter_number: 8
difficulty: 进阶
summary: Lazy Evaluation、Memoization、Tail Recursion、Trampolines、Continuation、Lens、Y Combinator
permalink: /books/cpp-functional-programming/08-advanced-patterns/
estimated_time: "90分钟"
github_exercises: "https://github.com/wzxzhuxi/cpp-functional-programming/tree/main/exercises"
github_solutions: "https://github.com/wzxzhuxi/cpp-functional-programming/tree/main/exercises/solutions"
---

## 说人话:高级模式是什么?

前面学了基础工具(纯函数、组合、Monad),现在学习如何组合这些工具解决复杂问题。

**本章内容**:
1. **Lazy Evaluation(惰性求值)**: 需要时才计算
2. **Memoization(记忆化)**: 缓存计算结果
3. **Tail Recursion(尾递归)**: 优化递归性能
4. **Trampolines(蹦床)**: 避免栈溢出
5. **Continuation(续延)**: 控制程序流程
6. **Lens(透镜)**: 优雅访问嵌套数据
7. **Y Combinator(Y 组合子)**: 匿名递归
8. **实际应用**: 组合这些模式解决真实问题

## 1. Lazy Evaluation(惰性求值)

### 什么是惰性求值?

**普通求值(eager)**: 立即计算
**惰性求值(lazy)**: 延迟到真正需要时才计算

### 为什么需要?

1. **避免不必要的计算**: 可能永远不会用到
2. **处理无限序列**: 按需生成
3. **提高性能**: 只计算需要的部分

### C++ 实现

```cpp
template<typename T>
class Lazy {
    mutable std::optional<T> cached;
    std::function<T()> thunk;

public:
    explicit Lazy(std::function<T()> f) : thunk(f) {}

    const T& force() const {
        if (!cached) {
            cached = thunk();
        }
        return *cached;
    }
};

// 使用
Lazy<int> expensive_calc([]() {
    std::cout << "Computing...\n";
    return 42 * 42;
});

// 此时还没计算
std::cout << "Before force\n";

// 现在才计算
int result = expensive_calc.force();  // 输出 "Computing..."

// 第二次直接用缓存
int result2 = expensive_calc.force();  // 不输出
```

### 应用:无限序列

```cpp
class InfiniteRange {
    int current;

public:
    InfiniteRange(int start = 0) : current(start) {}

    int next() { return current++; }

    // 惰性 map
    template<typename F>
    auto map(F f) {
        return [this, f]() { return f(next()); };
    }

    // 惰性 filter
    template<typename P>
    int next_if(P predicate) {
        while (true) {
            int val = next();
            if (predicate(val)) return val;
        }
    }
};

// 使用:无限的偶数序列
InfiniteRange range;
for (int i = 0; i < 10; ++i) {
    int even = range.next_if([](int x) { return x % 2 == 0; });
    std::cout << even << " ";
}
// 输出:0 2 4 6 8 10 12 14 16 18
```

## 2. Memoization(记忆化)

### 什么是记忆化?

缓存函数的计算结果,相同输入直接返回缓存。

### 实现

```cpp
template<typename R, typename... Args>
auto memoize(std::function<R(Args...)> func) {
    std::map<std::tuple<Args...>, R> cache;

    return [func, cache](Args... args) mutable -> R {
        auto key = std::make_tuple(args...);
        auto it = cache.find(key);
        if (it != cache.end()) {
            return it->second;
        }
        R result = func(args...);
        cache[key] = result;
        return result;
    };
}

// 使用:斐波那契数列
std::function<int(int)> fib = [&fib](int n) -> int {
    if (n <= 1) return n;
    return fib(n - 1) + fib(n - 2);
};

auto fib_memo = memoize(fib);

// O(n) 而不是 O(2^n)
int result = fib_memo(40);
```

## 3. Tail Recursion(尾递归)

### 什么是尾递归?

递归调用是函数的最后一个操作。编译器可以优化为循环,避免栈溢出。

### 非尾递归 vs 尾递归

```cpp
// 非尾递归:递归后还有操作(加法)
int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);  // 递归后还要乘 n
}

// 尾递归:递归是最后的操作
int factorial_tail(int n, int acc = 1) {
    if (n <= 1) return acc;
    return factorial_tail(n - 1, n * acc);  // 递归是最后操作
}
```

### 累加器技巧

将中间结果作为参数传递,避免递归后的操作。

```cpp
// 非尾递归求和
int sum(const std::vector<int>& vec, int index = 0) {
    if (index >= vec.size()) return 0;
    return vec[index] + sum(vec, index + 1);
}

// 尾递归求和
int sum_tail(const std::vector<int>& vec, int index = 0, int acc = 0) {
    if (index >= vec.size()) return acc;
    return sum_tail(vec, index + 1, acc + vec[index]);
}
```

## 4. Trampolines(蹦床)

### 什么是蹦床?

手动实现尾递归优化,将递归转换为循环。

### 为什么需要?

C++ 编译器不保证尾递归优化,大数据时可能栈溢出。

### 实现

```cpp
template<typename T>
struct Bounce {
    std::variant<T, std::function<Bounce<T>()>> data;

    bool is_done() const { return data.index() == 0; }
    T get_value() const { return std::get<0>(data); }
    Bounce<T> next() const { return std::get<1>(data)(); }

    static Bounce done(T value) {
        return Bounce{std::variant<T, std::function<Bounce<T>()>>(
            std::in_place_index<0>, value
        )};
    }

    static Bounce more(std::function<Bounce<T>()> f) {
        return Bounce{std::variant<T, std::function<Bounce<T>()>>(
            std::in_place_index<1>, f
        )};
    }
};

template<typename T>
T trampoline(Bounce<T> bounce) {
    while (!bounce.is_done()) {
        bounce = bounce.next();
    }
    return bounce.get_value();
}

// 使用:不会栈溢出的递归
Bounce<int> factorial_bounce(int n, int acc = 1) {
    if (n <= 1) {
        return Bounce<int>::done(acc);
    }
    return Bounce<int>::more([=]() {
        return factorial_bounce(n - 1, n * acc);
    });
}

int result = trampoline(factorial_bounce(100000));
```

## 5. Continuation(续延)

### 什么是续延?

将"接下来要做什么"作为参数传递。

### Continuation Passing Style (CPS)

```cpp
// 普通风格
int add(int a, int b) {
    return a + b;
}

// CPS 风格
void add_cps(int a, int b, std::function<void(int)> cont) {
    cont(a + b);
}

// 使用
add_cps(3, 4, [](int result) {
    std::cout << "Result: " << result << "\n";
});
```

### 为什么有用?

1. **控制流程**: 显式控制下一步做什么
2. **异步编程**: 回调就是 continuation
3. **异常处理**: 可以传递多个 continuation(成功、失败)

## 6. Lens(透镜)

### 什么是 Lens?

函数式方式访问和修改嵌套数据结构。

### 问题

```cpp
struct Address {
    std::string street;
    std::string city;
};

struct Person {
    std::string name;
    Address address;
};

// 修改嵌套数据很麻烦
Person update_city(const Person& p, const std::string& new_city) {
    Person new_p = p;
    new_p.address.city = new_city;
    return new_p;
}
```

### Lens 解决方案

```cpp
template<typename S, typename A>
struct Lens {
    std::function<A(const S&)> getter;
    std::function<S(const S&, const A&)> setter;

    A get(const S& s) const { return getter(s); }

    S set(const S& s, const A& a) const { return setter(s, a); }

    S modify(const S& s, std::function<A(const A&)> f) const {
        return set(s, f(get(s)));
    }

    // 组合 Lens
    template<typename B>
    Lens<S, B> compose(const Lens<A, B>& other) const {
        return Lens<S, B>{
            [this, other](const S& s) { return other.get(this->get(s)); },
            [this, other](const S& s, const B& b) {
                return this->set(s, other.set(this->get(s), b));
            }
        };
    }
};

// 定义 Lens
Lens<Person, Address> address_lens{
    [](const Person& p) { return p.address; },
    [](const Person& p, const Address& a) {
        Person new_p = p;
        new_p.address = a;
        return new_p;
    }
};

Lens<Address, std::string> city_lens{
    [](const Address& a) { return a.city; },
    [](const Address& a, const std::string& c) {
        Address new_a = a;
        new_a.city = c;
        return new_a;
    }
};

// 组合使用
auto person_city_lens = address_lens.compose(city_lens);
Person updated = person_city_lens.set(person, "New York");
```

## 7. Y Combinator(Y 组合子)

### 什么是 Y Combinator?

在没有命名递归的情况下实现递归。

### 实现

```cpp
template<typename F>
struct Y {
    F f;

    template<typename... Args>
    auto operator()(Args&&... args) const {
        return f(*this, std::forward<Args>(args)...);
    }
};

template<typename F>
Y<std::decay_t<F>> make_y(F&& f) {
    return {std::forward<F>(f)};
}

// 使用:匿名递归
auto factorial = make_y([](auto self, int n) -> int {
    if (n <= 1) return 1;
    return n * self(n - 1);
});

int result = factorial(5);  // 120
```

## 8. 模式对比

| 模式 | 用途 | 优点 | 缺点 |
|------|------|------|------|
| Lazy Evaluation | 延迟计算 | 性能优化,处理无限序列 | 调试困难 |
| Memoization | 缓存结果 | 避免重复计算 | 内存开销 |
| Tail Recursion | 优化递归 | 避免栈溢出 | 需要改写算法 |
| Trampoline | 手动优化递归 | 确保不栈溢出 | 代码复杂 |
| Continuation | 控制流程 | 灵活的异步处理 | 难以理解 |
| Lens | 访问嵌套数据 | 不可变更新优雅 | 模板代码多 |
| Y Combinator | 匿名递归 | 理论优雅 | 实际用处有限 |

## 9. 何时使用这些模式?

### Lazy Evaluation
- 处理大数据集
- 可能不需要全部数据
- 构建无限序列

### Memoization
- 纯函数且计算昂贵
- 相同输入会重复调用
- 输入空间不是无限大

### Tail Recursion / Trampoline
- 递归深度很大
- 担心栈溢出
- 可以改写成尾递归形式

### Continuation
- 异步编程
- 需要精确控制流程
- 实现协程或生成器

### Lens
- 频繁更新嵌套不可变数据
- 需要组合多个访问路径
- 数据结构层次深

## 10. 最佳实践

1. **不要过度使用**: 这些模式很强大,但不是银弹
2. **先简单后复杂**: 先用简单方法,遇到问题再优化
3. **性能测试**: 抽象有代价,确保值得
4. **文档注释**: 这些模式不直观,需要解释
5. **团队共识**: 确保团队理解这些模式

## 11. 性能考虑

### Lazy Evaluation
- **优势**: 避免不必要的计算
- **劣势**: thunk 的函数调用开销

### Memoization
- **优势**: O(1) 查找缓存
- **劣势**: 内存占用,可能需要 LRU 策略

### Trampoline
- **优势**: 避免栈溢出
- **劣势**: 堆分配开销,比真正的尾递归优化慢

## 12. 调试技巧

1. **Lazy Evaluation**: 添加日志记录计算时机
2. **Memoization**: 记录缓存命中率
3. **Trampoline**: 限制最大跳转次数,防止死循环
4. **Continuation**: 打印 continuation 链

## 总结

高级函数式模式是工具箱中的高级工具:

- **Lazy Evaluation**: 延迟到需要时
- **Memoization**: 记住计算过的
- **Tail Recursion**: 递归转循环
- **Trampoline**: 手动优化递归
- **Continuation**: 控制下一步
- **Lens**: 优雅访问嵌套
- **Y Combinator**: 匿名递归

这些模式不是必须的,但在合适的场景下能显著提升代码质量。关键是理解每个模式解决什么问题,而不是盲目使用。

---

## 完整代码示例

本节包含完整可运行示例,涵盖所有核心概念:

### 1. Lazy Evaluation 示例

```cpp
template<typename T>
class Lazy {
    mutable std::optional<T> cached;
    std::function<T()> thunk;

public:
    explicit Lazy(std::function<T()> f) : thunk(f) {}

    const T& force() const {
        if (!cached) {
            cached = thunk();
        }
        return *cached;
    }

    bool is_evaluated() const { return cached.has_value(); }
};

void lazy_evaluation_demo() {
    std::cout << "=== Lazy Evaluation ===\n";

    Lazy<int> expensive_calc([]() {
        std::cout << "Computing expensive calculation...\n";
        int sum = 0;
        for (int i = 0; i < 1000; ++i) {
            sum += i;
        }
        return sum;
    });

    std::cout << "Lazy value created (not computed yet)\n";
    std::cout << "Is evaluated: " << (expensive_calc.is_evaluated() ? "yes" : "no") << "\n";

    std::cout << "\nFirst force:\n";
    int result1 = expensive_calc.force();
    std::cout << "Result: " << result1 << "\n";
    std::cout << "Is evaluated: " << (expensive_calc.is_evaluated() ? "yes" : "no") << "\n";

    std::cout << "\nSecond force (using cache):\n";
    int result2 = expensive_calc.force();
    std::cout << "Result: " << result2 << "\n\n";
}
```

**要点**:
- `mutable std::optional<T>` 允许在 const 方法中修改缓存
- `thunk` 是封装了计算的无参数函数
- 第一次 `force()` 触发计算并缓存结果
- 后续调用直接返回缓存,避免重复计算

### 2. Infinite Range 示例

```cpp
class InfiniteRange {
    int current;

public:
    explicit InfiniteRange(int start = 0) : current(start) {}

    int next() { return current++; }

    template<typename P>
    int next_if(P predicate) {
        while (true) {
            int val = next();
            if (predicate(val)) return val;
        }
    }

    std::vector<int> take(int n) {
        std::vector<int> result;
        for (int i = 0; i < n; ++i) {
            result.push_back(next());
        }
        return result;
    }

    template<typename P>
    std::vector<int> take_while(int max, P predicate) {
        std::vector<int> result;
        for (int i = 0; i < max; ++i) {
            int val = next_if(predicate);
            result.push_back(val);
        }
        return result;
    }
};

void infinite_range_demo() {
    std::cout << "=== Infinite Range ===\n";

    InfiniteRange range1(0);
    auto first_10 = range1.take(10);
    std::cout << "First 10 numbers: ";
    for (int n : first_10) {
        std::cout << n << " ";
    }
    std::cout << "\n";

    InfiniteRange range2(0);
    auto first_10_evens = range2.take_while(10, [](int x) { return x % 2 == 0; });
    std::cout << "First 10 even numbers: ";
    for (int n : first_10_evens) {
        std::cout << n << " ";
    }
    std::cout << "\n";

    InfiniteRange range3(0);
    auto first_10_div3 = range3.take_while(10, [](int x) { return x % 3 == 0; });
    std::cout << "First 10 numbers divisible by 3: ";
    for (int n : first_10_div3) {
        std::cout << n << " ";
    }
    std::cout << "\n\n";
}
```

**要点**:
- 无限序列不预先生成所有元素,按需生成
- `next_if` 跳过不满足条件的元素
- `take(n)` 获取前 n 个元素
- 适合处理大数据流,内存效率高

### 3. Memoization 示例

```cpp
void memoization_demo() {
    std::cout << "=== Memoization ===\n";

    std::map<int, long long> fib_cache;
    std::function<long long(int)> fib_cached = [&](int n) -> long long {
        if (n <= 1) return n;
        auto it = fib_cache.find(n);
        if (it != fib_cache.end()) return it->second;
        long long result = fib_cached(n - 1) + fib_cached(n - 2);
        fib_cache[n] = result;
        return result;
    };

    std::cout << "Fibonacci(30) with memoization: " << fib_cached(30) << "\n";
    std::cout << "Fibonacci(40) with memoization: " << fib_cached(40) << "\n";
    std::cout << "Cache size: " << fib_cache.size() << "\n\n";
}
```

**要点**:
- 使用 `std::map` 存储参数到结果的映射
- 查找缓存优先于计算,命中则直接返回
- 将指数时间复杂度降低到线性
- 适合纯函数且重复调用的场景

### 4. Tail Recursion 示例

```cpp
int factorial_normal(int n) {
    if (n <= 1) return 1;
    return n * factorial_normal(n - 1);
}

int factorial_tail(int n, int acc = 1) {
    if (n <= 1) return acc;
    return factorial_tail(n - 1, n * acc);
}

int sum_normal(const std::vector<int>& vec, size_t index = 0) {
    if (index >= vec.size()) return 0;
    return vec[index] + sum_normal(vec, index + 1);
}

int sum_tail(const std::vector<int>& vec, size_t index = 0, int acc = 0) {
    if (index >= vec.size()) return acc;
    return sum_tail(vec, index + 1, acc + vec[index]);
}

void tail_recursion_demo() {
    std::cout << "=== Tail Recursion ===\n";

    std::cout << "Factorial(10) normal: " << factorial_normal(10) << "\n";
    std::cout << "Factorial(10) tail: " << factorial_tail(10) << "\n";

    std::vector<int> nums{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    std::cout << "Sum normal: " << sum_normal(nums) << "\n";
    std::cout << "Sum tail: " << sum_tail(nums) << "\n\n";
}
```

**要点**:
- 累加器参数 `acc` 保存中间计算结果
- 避免递归返回后的额外操作
- 编译器可以将尾递归优化为循环(TCO)
- 大幅降低栈溢出风险

### 5. Trampoline 示例

```cpp
template<typename T>
struct Bounce {
    std::variant<T, std::function<Bounce<T>()>> data;

    bool is_done() const { return data.index() == 0; }
    T get_value() const { return std::get<0>(data); }
    Bounce<T> next() const { return std::get<1>(data)(); }

    static Bounce done(T value) {
        return Bounce{std::variant<T, std::function<Bounce<T>()>>(
            std::in_place_index<0>, value
        )};
    }

    static Bounce more(std::function<Bounce<T>()> f) {
        return Bounce{std::variant<T, std::function<Bounce<T>()>>(
            std::in_place_index<1>, f
        )};
    }
};

template<typename T>
T trampoline(Bounce<T> bounce) {
    while (!bounce.is_done()) {
        bounce = bounce.next();
    }
    return bounce.get_value();
}

Bounce<long long> factorial_bounce(int n, long long acc = 1) {
    if (n <= 1) {
        return Bounce<long long>::done(acc);
    }
    return Bounce<long long>::more([=]() {
        return factorial_bounce(n - 1, n * acc);
    });
}

Bounce<int> sum_bounce(const std::vector<int>& vec, size_t index = 0, int acc = 0) {
    if (index >= vec.size()) {
        return Bounce<int>::done(acc);
    }
    return Bounce<int>::more([&vec, index, acc]() {
        return sum_bounce(vec, index + 1, acc + vec[index]);
    });
}

void trampoline_demo() {
    std::cout << "=== Trampoline ===\n";

    long long result1 = trampoline(factorial_bounce(20));
    std::cout << "Factorial(20) with trampoline: " << result1 << "\n";

    std::vector<int> nums{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    int result2 = trampoline(sum_bounce(nums));
    std::cout << "Sum with trampoline: " << result2 << "\n\n";
}
```

**要点**:
- `Bounce<T>` 表示"完成"或"继续计算"两种状态
- `done(value)` 返回最终结果,终止计算
- `more(function)` 返回下一步的计算函数
- `trampoline` 循环驱动,避免递归栈增长
- 不依赖编译器优化,手动保证不栈溢出

### 6. Continuation Passing Style 示例

```cpp
int add_normal(int a, int b) {
    return a + b;
}

void add_cps(int a, int b, std::function<void(int)> cont) {
    cont(a + b);
}

void calculate_cps(int x, std::function<void(int)> cont) {
    add_cps(x, 10, [cont](int r1) {
        add_cps(r1, 20, [cont](int r2) {
            add_cps(r2, 30, cont);
        });
    });
}

void continuation_demo() {
    std::cout << "=== Continuation Passing Style ===\n";

    std::cout << "Normal add: " << add_normal(3, 4) << "\n";

    std::cout << "CPS add: ";
    add_cps(3, 4, [](int result) {
        std::cout << result << "\n";
    });

    std::cout << "CPS chain (5 + 10 + 20 + 30): ";
    calculate_cps(5, [](int result) {
        std::cout << result << "\n";
    });
    std::cout << "\n";
}
```

**要点**:
- CPS 函数不返回值,而是调用 continuation 传递结果
- Continuation 封装了"接下来要做的事情"
- 可以嵌套 continuation 形成计算链
- 是异步编程(Promise/callback)的理论基础

### 7. Lens 示例

```cpp
template<typename S, typename A>
struct Lens {
    std::function<A(const S&)> getter;
    std::function<S(const S&, const A&)> setter;

    A get(const S& s) const { return getter(s); }

    S set(const S& s, const A& a) const { return setter(s, a); }

    S modify(const S& s, std::function<A(const A&)> f) const {
        return set(s, f(get(s)));
    }

    template<typename B>
    Lens<S, B> compose(const Lens<A, B>& other) const {
        return Lens<S, B>{
            [this, other](const S& s) { return other.get(this->get(s)); },
            [this, other](const S& s, const B& b) {
                return this->set(s, other.set(this->get(s), b));
            }
        };
    }
};

struct Address {
    std::string street;
    std::string city;
    int zipcode;
};

struct Person {
    std::string name;
    int age;
    Address address;
};

void lens_demo() {
    std::cout << "=== Lens ===\n";

    Lens<Person, std::string> name_lens{
        [](const Person& p) { return p.name; },
        [](const Person& p, const std::string& n) {
            Person new_p = p;
            new_p.name = n;
            return new_p;
        }
    };

    Lens<Person, Address> address_lens{
        [](const Person& p) { return p.address; },
        [](const Person& p, const Address& a) {
            Person new_p = p;
            new_p.address = a;
            return new_p;
        }
    };

    Lens<Address, std::string> city_lens{
        [](const Address& a) { return a.city; },
        [](const Address& a, const std::string& c) {
            Address new_a = a;
            new_a.city = c;
            return new_a;
        }
    };

    Lens<Address, int> zipcode_lens{
        [](const Address& a) { return a.zipcode; },
        [](const Address& a, int z) {
            Address new_a = a;
            new_a.zipcode = z;
            return new_a;
        }
    };

    Person alice{"Alice", 30, {"123 Main St", "Boston", 02101}};

    std::cout << "Original: " << alice.name << " lives in "
              << alice.address.city << " " << alice.address.zipcode << "\n";

    Person alice2 = name_lens.set(alice, "Alice Smith");
    std::cout << "After name change: " << alice2.name << "\n";

    auto person_city_lens = address_lens.compose(city_lens);
    Person alice3 = person_city_lens.set(alice, "New York");
    std::cout << "After city change: " << alice3.address.city << "\n";

    auto person_zipcode_lens = address_lens.compose(zipcode_lens);
    Person alice4 = person_zipcode_lens.modify(alice, [](int z) { return z + 1000; });
    std::cout << "After zipcode modify: " << alice4.address.zipcode << "\n\n";
}
```

**要点**:
- Lens 由 getter 和 setter 两个函数组成
- `get(s)` 从整体提取部分,`set(s, a)` 用新部分创建新整体
- `modify(s, f)` 应用函数变换部分
- `compose` 组合多个 Lens 访问深层嵌套字段
- 保持不可变性,所有操作返回新对象

### 8. Y Combinator 示例

```cpp
template<typename F>
struct Y {
    F f;

    template<typename... Args>
    auto operator()(Args&&... args) const {
        return f(*this, std::forward<Args>(args)...);
    }
};

template<typename F>
Y<std::decay_t<F>> make_y(F&& f) {
    return {std::forward<F>(f)};
}

void y_combinator_demo() {
    std::cout << "=== Y Combinator ===\n";

    auto my_factorial = make_y([](auto self, int n) -> long long {
        if (n <= 1) return 1;
        return n * self(n - 1);
    });

    std::cout << "Factorial(10) with Y: " << my_factorial(10) << "\n";

    auto my_fib = make_y([](auto self, int n) -> long long {
        if (n <= 1) return n;
        return self(n - 1) + self(n - 2);
    });

    std::cout << "Fibonacci(15) with Y: " << my_fib(15) << "\n";

    auto my_sum = make_y([](auto self, const std::vector<int>& vec, size_t index) -> int {
        if (index >= vec.size()) return 0;
        return vec[index] + self(vec, index + 1);
    });

    std::vector<int> nums{1, 2, 3, 4, 5};
    std::cout << "Sum with Y: " << my_sum(nums, 0) << "\n\n";
}
```

**要点**:
- Y 组合子实现匿名函数的递归
- `Y` 结构体将自身作为第一个参数传递给函数
- Lambda 通过 `self` 参数递归调用自己
- 展示了函数作为一等公民的强大表达力
- 理论优雅,但实际应用有限

### 运行完整示例

```cpp
int main() {
    lazy_evaluation_demo();
    infinite_range_demo();
    memoization_demo();
    tail_recursion_demo();
    trampoline_demo();
    continuation_demo();
    lens_demo();
    y_combinator_demo();

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
./bin/08_advanced_patterns
```

**GitHub 完整源码**: [08_advanced_patterns.cpp](https://github.com/wzxzhuxi/cpp-functional-programming/blob/main/examples/advanced/08_advanced_patterns.cpp)

### 示例输出

```
=== Lazy Evaluation ===
Lazy value created (not computed yet)
Is evaluated: no

First force:
Computing expensive calculation...
Result: 499500
Is evaluated: yes

Second force (using cache):
Result: 499500

=== Infinite Range ===
First 10 numbers: 0 1 2 3 4 5 6 7 8 9
First 10 even numbers: 0 2 4 6 8 10 12 14 16 18
First 10 numbers divisible by 3: 0 3 6 9 12 15 18 21 24 27

=== Memoization ===
Fibonacci(30) with memoization: 832040
Fibonacci(40) with memoization: 102334155
Cache size: 41

=== Tail Recursion ===
Factorial(10) normal: 3628800
Factorial(10) tail: 3628800
Sum normal: 55
Sum tail: 55

=== Trampoline ===
Factorial(20) with trampoline: 2432902008176640000
Sum with trampoline: 55

=== Continuation Passing Style ===
Normal add: 7
CPS add: 7
CPS chain (5 + 10 + 20 + 30): 65

=== Lens ===
Original: Alice lives in Boston 2101
After name change: Alice Smith
After city change: New York
After zipcode modify: 3101

=== Y Combinator ===
Factorial(10) with Y: 3628800
Fibonacci(15) with Y: 610
Sum with Y: 15
```

### 关键要点总结

1. **Lazy Evaluation 延迟计算**: 只在真正需要时触发计算,缓存结果避免重复
2. **Memoization 记忆化**: 用空间换时间,将重复计算从 O(2^n) 降至 O(n)
3. **Tail Recursion 尾递归**: 累加器模式将递归优化为循环,避免栈溢出
4. **Trampoline 蹦床**: 手动实现尾递归优化,确保不依赖编译器即可安全递归
5. **Continuation 续延**: 将控制流显式化,是异步编程和协程的理论基础
6. **Lens 透镜**: 组合式访问嵌套数据,优雅处理不可变更新
7. **Y Combinator Y 组合子**: 匿名递归的理论实现,展示函数式编程的表达力
8. **性能与可维护性平衡**: 这些模式强大但复杂,按需使用,过度抽象有害

---

## 练习题

本章提供了配套的练习题,帮助你巩固所学知识:

1. 实现惰性列表(Lazy List),支持 map、filter、take 操作
2. 为任意二元函数实现通用 memoization 包装器
3. 将普通递归的二叉树遍历改写为尾递归形式
4. 用 Trampoline 实现安全的深层嵌套 JSON 解析
5. 实现支持 Lens 的不可变配置树
6. 用 Y Combinator 实现快速排序

**[查看练习题 →](https://github.com/wzxzhuxi/cpp-functional-programming/blob/main/exercises/08_advanced_patterns_exercises.cpp)**

练习题包含:
- 详细的问题描述
- 测试用例和预期输出
- 难度分级和提示

### 参考答案

⚠️ **建议先独立完成练习再查看答案**

**[查看参考答案 →](https://github.com/wzxzhuxi/cpp-functional-programming/blob/main/exercises/solutions/08_advanced_patterns_solutions.cpp)**

答案包含:
- 完整的实现代码
- 详细的解题思路
- 常见错误分析
