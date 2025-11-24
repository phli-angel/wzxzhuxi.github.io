---
title: 第七章:Monad 模式
book: cpp-functional-programming
order: 7
chapter_number: 7
difficulty: 进阶
summary: Maybe/Optional、Result/Either、List、IO Monad 及其实际应用
permalink: /books/cpp-functional-programming/07-monads/
estimated_time: "75分钟"
github_exercises: "https://github.com/wzxzhuxi/cpp-functional-programming/tree/main/exercises"
github_solutions: "https://github.com/wzxzhuxi/cpp-functional-programming/tree/main/exercises/solutions"
---

## 说人话:Monad 是什么?

Monad 不是什么神秘的魔法。**Monad 就是一个带有特定操作的容器**,让你能够:
1. 把值放进容器(`return` / `pure`)
2. 对容器里的值进行连续操作,自动处理容器的"副作用"(`bind` / `>>=`)

**核心思想**:把"如何处理包装"的逻辑从业务逻辑中分离出来。

### 类比

想象你在流水线上处理包裹:
- **普通函数**:`int -> int`(拿裸露的物品,返回裸露的物品)
- **Monad**:`Box<int> -> Box<int>`(拿包裹,返回包裹,自动处理包装/拆包装)

## 1. Monad 的三大定律

每个 Monad 必须满足:

### 定律 1:左单位元(Left Identity)

```cpp
return(a).bind(f) == f(a)
```

**含义**:先包装再操作 = 直接操作

### 定律 2:右单位元(Right Identity)

```cpp
m.bind(return) == m
```

**含义**:操作后包装回去 = 什么都不做

### 定律 3:结合律(Associativity)

```cpp
m.bind(f).bind(g) == m.bind(x => f(x).bind(g))
```

**含义**:链式操作的顺序无关紧要

## 2. Optional/Maybe Monad

处理"可能没有值"的情况。

### 实现

```cpp
template<typename T>
class Maybe {
public:
    std::optional<T> value;

    // return / pure
    static Maybe just(T v) {
        return Maybe{std::optional<T>(v)};
    }

    static Maybe nothing() {
        return Maybe{std::nullopt};
    }

    // bind / >>=
    template<typename F>
    auto bind(F f) const {
        using R = decltype(f(std::declval<T>()));
        if (!value) return R::nothing();
        return f(*value);
    }

    // fmap (Functor)
    template<typename F>
    auto fmap(F f) const {
        using NewT = decltype(f(std::declval<T>()));
        if (!value) return Maybe<NewT>::nothing();
        return Maybe<NewT>::just(f(*value));
    }
};
```

### 使用案例

```cpp
// 不使用 Monad(丑陋的嵌套)
std::optional<int> divide(int a, int b) {
    if (b == 0) return std::nullopt;
    return a / b;
}

std::optional<int> safe_sqrt(int x) {
    if (x < 0) return std::nullopt;
    return std::sqrt(x);
}

auto result = divide(100, 5);
if (result) {
    auto result2 = safe_sqrt(*result);
    if (result2) {
        // 使用 result2
    }
}

// 使用 Monad(清晰的链式)
Maybe<int>::just(100)
    .bind(divide_maybe(5))
    .bind(safe_sqrt_maybe);
```

## 3. Result/Either Monad

处理"可能失败"的情况,携带错误信息。

### 实现

```cpp
template<typename T, typename E>
class Result {
public:
    std::variant<T, E> data;

    bool is_ok() const { return std::holds_alternative<T>(data); }
    const T& unwrap() const { return std::get<T>(data); }
    const E& unwrap_err() const { return std::get<E>(data); }

    // return / pure
    static Result ok(T value) {
        return Result{std::variant<T, E>(std::in_place_index<0>, value)};
    }

    static Result err(E error) {
        return Result{std::variant<T, E>(std::in_place_index<1>, error)};
    }

    // bind / >>=
    template<typename F>
    auto bind(F f) const {
        using R = decltype(f(std::declval<T>()));
        if (!is_ok()) {
            return R::err(unwrap_err());
        }
        return f(unwrap());
    }

    // fmap
    template<typename F>
    auto fmap(F f) const {
        using NewT = decltype(f(std::declval<T>()));
        if (!is_ok()) {
            return Result<NewT, E>::err(unwrap_err());
        }
        return Result<NewT, E>::ok(f(unwrap()));
    }
};
```

### 使用案例:错误传播

```cpp
Result<int, std::string> parse_int(const std::string& s);
Result<double, std::string> to_celsius(int fahrenheit);
Result<std::string, std::string> format_temp(double celsius);

// 链式调用,错误自动传播
auto result = parse_int("32")
    .bind(to_celsius)
    .bind(format_temp);

if (result.is_ok()) {
    std::cout << result.unwrap();
} else {
    std::cout << "Error: " << result.unwrap_err();
}
```

## 4. IO Monad 概念

处理副作用(I/O 操作)。

### 为什么需要 IO Monad?

纯函数不应该有副作用,但程序必须与外界交互。IO Monad 将副作用"包装"起来,标记哪些操作是不纯的。

### 简化实现(概念演示)

```cpp
template<typename T>
class IO {
public:
    std::function<T()> action;

    explicit IO(std::function<T()> f) : action(f) {}

    // return / pure
    static IO pure(T value) {
        return IO([value]() { return value; });
    }

    // bind
    template<typename F>
    auto bind(F f) const {
        return IO([action = this->action, f]() {
            T result = action();
            return f(result).action();
        });
    }

    // 执行副作用
    T run() const {
        return action();
    }
};

// 使用
IO<std::string> read_line() {
    return IO<std::string>([]() {
        std::string line;
        std::getline(std::cin, line);
        return line;
    });
}

IO<void> print(const std::string& s) {
    return IO<void>([s]() {
        std::cout << s << "\n";
    });
}

// 链式 I/O 操作
auto program = read_line()
    .bind([](const std::string& input) {
        return print("You entered: " + input);
    });

program.run();  // 执行副作用
```

## 5. List Monad

处理"多个值"的情况。

### 实现

{% raw %}
```cpp
template<typename T>
class List {
public:
    std::vector<T> items;

    static List pure(T value) {
        return List{std::vector<T>{value}};
    }

    template<typename F>
    auto bind(F f) const {
        using R = decltype(f(std::declval<T>()));
        using NewT = typename R::value_type;

        std::vector<NewT> result;
        for (const auto& item : items) {
            auto r = f(item);
            result.insert(result.end(), r.items.begin(), r.items.end());
        }
        return List<NewT>{result};
    }
};

// 使用:生成所有组合
List<int> numbers{{1, 2, 3}};
List<char> letters{{'a', 'b'}};

auto combinations = numbers.bind([&](int n) {
    return letters.bind([n](char c) {
        return List<std::pair<int, char>>::pure({n, c});
    });
});
// 结果:(1,a), (1,b), (2,a), (2,b), (3,a), (3,b)
```
{% endraw %}

## 6. Monad vs Functor vs Applicative

### Functor(映射器)

只有 `fmap`:
```cpp
fmap :: (a -> b) -> f a -> f b
```

**含义**:对容器里的值应用函数。

### Applicative(应用器)

有 `pure` 和 `ap`:
```cpp
pure :: a -> f a
ap :: f (a -> b) -> f a -> f b
```

**含义**:容器里的函数应用到容器里的值。

### Monad(单子)

有 `return` 和 `bind`:
```cpp
return :: a -> m a
bind :: m a -> (a -> m b) -> m b
```

**含义**:值到容器的函数,可以链式调用。

### 关系

```
Functor < Applicative < Monad
```

每个 Monad 都是 Applicative,每个 Applicative 都是 Functor。

## 7. 实际应用

### 7.1 数据库查询链

```cpp
Result<User, DbError> find_user(int id);
Result<Orders, DbError> find_orders(const User& user);
Result<double, DbError> calculate_total(const Orders& orders);

auto total = find_user(123)
    .bind(find_orders)
    .bind(calculate_total);
```

### 7.2 解析器组合

```cpp
Parser<int> parse_int();
Parser<std::string> parse_string();

auto parser = parse_int()
    .bind([](int n) {
        return parse_string().bind([n](std::string s) {
            return Parser<Pair>::pure({n, s});
        });
    });
```

### 7.3 异步操作链

```cpp
Future<Response> fetch(const std::string& url);
Future<Data> parse(const Response& resp);
Future<void> save(const Data& data);

auto pipeline = fetch("https://api.example.com")
    .bind(parse)
    .bind(save);
```

## 8. 为什么 Monad 有用?

### 1. 错误处理

不需要到处写 `if (error)`,错误自动传播。

### 2. 副作用管理

明确标记哪些代码有副作用(IO Monad)。

### 3. 状态传递

隐式传递状态(State Monad)。

### 4. 代码组合

小的 monadic 函数可以轻松组合成大的。

## 9. Monad 的坏名声

Monad 在初学者中名声不好,因为:

1. **过度抽象的教学**:用范畴论解释,吓跑新人
2. **术语晦涩**:functor、applicative、endofunctor...
3. **实用价值不明显**:小程序看不出优势

**真相**:
- Monad 就是设计模式,和工厂模式、观察者模式一样
- 不需要数学背景
- 用多了就自然理解了

---

## 最佳实践

1. **不要过度使用**:不是所有代码都要 monadic
2. **从简单开始**:先用 Optional/Result,再考虑其他
3. **实用主义**:能用简单方法解决就不要强行 Monad
4. **命名清晰**:`bind`、`fmap`、`pure` 等术语要统一

## 为什么 Monad 有用?

### 1. 错误处理
不需要到处写 `if (error)`,错误自动传播。

### 2. 副作用管理
明确标记哪些代码有副作用(IO Monad)。

### 3. 状态传递
隐式传递状态(State Monad)。

### 4. 代码组合
小的 monadic 函数可以轻松组合成大的。

## 总结

Monad 的本质:
- **包装值**:把值放进容器
- **链式操作**:自动处理容器逻辑
- **关注业务**:不用关心包装/拆包装

常见 Monad:
- **Maybe/Optional**:处理可能没有值
- **Result/Either**:处理可能失败
- **IO**:处理副作用
- **List**:处理多个值

掌握 Monad,代码会更简洁、更安全、更易组合。

---

## 完整代码示例

本节包含完整可运行示例,涵盖所有核心概念:

### 1. Maybe Monad 示例

```cpp
template<typename T>
class Maybe {
public:
    std::optional<T> value;

    // Constructors
    Maybe() : value(std::nullopt) {}
    explicit Maybe(T v) : value(v) {}
    explicit Maybe(std::nullopt_t) : value(std::nullopt) {}

    // return / pure
    static Maybe just(T v) {
        return Maybe(v);
    }

    static Maybe nothing() {
        return Maybe(std::nullopt);
    }

    // bind / >>=
    template<typename F>
    auto bind(F f) const {
        using R = decltype(f(std::declval<T>()));
        if (!value) return R::nothing();
        return f(*value);
    }

    // fmap (Functor)
    template<typename F>
    auto fmap(F f) const {
        using NewT = decltype(f(std::declval<T>()));
        if (!value) return Maybe<NewT>::nothing();
        return Maybe<NewT>::just(f(*value));
    }

    // has_value
    bool has_value() const { return value.has_value(); }

    // get
    const T& get() const { return *value; }
};

// Maybe 辅助函数
Maybe<int> safe_divide(int a, int b) {
    if (b == 0) return Maybe<int>::nothing();
    return Maybe<int>::just(a / b);
}

Maybe<double> safe_sqrt(double x) {
    if (x < 0) return Maybe<double>::nothing();
    return Maybe<double>::just(std::sqrt(x));
}

void maybe_monad_demo() {
    std::cout << "=== Maybe Monad ===\n";

    // 成功的链式调用
    auto result1 = Maybe<int>::just(100)
        .bind([](int x) { return safe_divide(x, 5); })
        .bind([](int x) { return Maybe<double>::just(static_cast<double>(x)); })
        .bind(safe_sqrt);

    if (result1.has_value()) {
        std::cout << "100 / 5, then sqrt: " << result1.get() << "\n";
    }

    // 中途失败的链式调用
    auto result2 = Maybe<int>::just(100)
        .bind([](int x) { return safe_divide(x, 0); })  // 失败
        .bind([](int x) { return Maybe<double>::just(static_cast<double>(x)); })
        .bind(safe_sqrt);

    std::cout << "100 / 0 (failed): "
              << (result2.has_value() ? "Success" : "Nothing") << "\n";

    // 使用 fmap
    auto result3 = Maybe<int>::just(42)
        .fmap([](int x) { return x * 2; })
        .fmap([](int x) { return x + 10; });

    std::cout << "42 * 2 + 10 = " << result3.get() << "\n\n";
}
```

**要点**:
- `just` 包装值,`nothing` 表示无值
- `bind` 链式调用,中途失败则短路,跳过后续操作
- `fmap` 只对有值的 Maybe 应用函数,无值则传递
- 避免嵌套 if 检查,代码更清晰

### 2. Result Monad 示例

```cpp
template<typename T, typename E>
class Result {
public:
    std::variant<T, E> data;

    bool is_ok() const { return data.index() == 0; }
    bool is_err() const { return data.index() == 1; }
    const T& unwrap() const { return std::get<0>(data); }
    const E& unwrap_err() const { return std::get<1>(data); }

    // return / pure
    static Result ok(T value) {
        return Result{std::variant<T, E>(std::in_place_index<0>, value)};
    }

    static Result err(E error) {
        return Result{std::variant<T, E>(std::in_place_index<1>, error)};
    }

    // bind / >>=
    template<typename F>
    auto bind(F f) const {
        using R = decltype(f(std::declval<T>()));
        if (!is_ok()) {
            return R::err(unwrap_err());
        }
        return f(unwrap());
    }

    // fmap
    template<typename F>
    auto fmap(F f) const {
        using NewT = decltype(f(std::declval<T>()));
        if (!is_ok()) {
            return Result<NewT, E>::err(unwrap_err());
        }
        return Result<NewT, E>::ok(f(unwrap()));
    }
};

// Result 辅助函数
Result<int, std::string> parse_int(const std::string& s) {
    try {
        return Result<int, std::string>::ok(std::stoi(s));
    } catch (...) {
        return Result<int, std::string>::err("Invalid integer: " + s);
    }
}

Result<double, std::string> to_celsius(int fahrenheit) {
    return Result<double, std::string>::ok((fahrenheit - 32) * 5.0 / 9.0);
}

Result<std::string, std::string> format_temp(double celsius) {
    return Result<std::string, std::string>::ok(
        std::to_string(static_cast<int>(celsius)) + "°C"
    );
}

void result_monad_demo() {
    std::cout << "=== Result Monad ===\n";

    // 成功的转换链
    auto result1 = parse_int("77")
        .bind(to_celsius)
        .bind(format_temp);

    if (result1.is_ok()) {
        std::cout << "77°F = " << result1.unwrap() << "\n";
    }

    // 失败的转换链
    auto result2 = parse_int("invalid")
        .bind(to_celsius)
        .bind(format_temp);

    if (result2.is_err()) {
        std::cout << "Error: " << result2.unwrap_err() << "\n";
    }

    // 使用 fmap
    auto result3 = Result<int, std::string>::ok(10)
        .fmap([](int x) { return x * 2; })
        .fmap([](int x) { return x + 5; });

    std::cout << "10 * 2 + 5 = " << result3.unwrap() << "\n\n";
}
```

**要点**:
- `ok` 包装成功值,`err` 包装错误信息
- `bind` 自动传播错误,遇到 err 则短路
- 比抛异常更显式,错误类型在类型签名中
- 避免 try-catch 嵌套,错误处理集中在链末端

### 3. List Monad 示例

```cpp
template<typename T>
class List {
public:
    std::vector<T> items;

    List() = default;
    List(std::vector<T> v) : items(v) {}

    // return / pure
    static List pure(T value) {
        return List{std::vector<T>{value}};
    }

    // bind / >>=
    template<typename F>
    auto bind(F f) const {
        using R = decltype(f(std::declval<T>()));
        using NewT = typename decltype(R::items)::value_type;

        std::vector<NewT> result;
        for (const auto& item : items) {
            auto r = f(item);
            result.insert(result.end(), r.items.begin(), r.items.end());
        }
        return List<NewT>{result};
    }

    // fmap
    template<typename F>
    auto fmap(F f) const {
        using NewT = decltype(f(std::declval<T>()));
        std::vector<NewT> result;
        result.reserve(items.size());
        for (const auto& item : items) {
            result.push_back(f(item));
        }
        return List<NewT>{result};
    }
};

void list_monad_demo() {
    std::cout << "=== List Monad ===\n";

    // 生成所有组合
    {% raw %}List<int> numbers{{1, 2, 3}};
    List<std::string> letters{{"a", "b"}};{% endraw %}

    auto combinations = numbers.bind([&letters](int n) {
        return letters.fmap([n](const std::string& s) {
            return std::to_string(n) + s;
        });
    });

    std::cout << "Combinations: ";
    for (const auto& item : combinations.items) {
        std::cout << item << " ";
    }
    std::cout << "\n";

    // List 的 flatMap 效果
    {% raw %}List<int> nums{{1, 2, 3}};
    auto expanded = nums.bind([](int x) {
        return List<int>{{x, x * 10, x * 100}};
    });{% endraw %}

    std::cout << "Expanded: ";
    for (int x : expanded.items) {
        std::cout << x << " ";
    }
    std::cout << "\n\n";
}
```

**要点**:
- List Monad 的 `bind` 相当于 flatMap
- 对每个元素应用函数,然后扁平化结果
- 适合生成笛卡尔积、组合等场景
- 非确定性计算的建模

### 4. IO Monad 示例

```cpp
template<typename T>
class IO {
public:
    std::function<T()> action;

    explicit IO(std::function<T()> f) : action(f) {}

    // return / pure
    static IO pure(T value) {
        return IO([value]() { return value; });
    }

    // bind
    template<typename F>
    auto bind(F f) const -> decltype(f(std::declval<T>())) {
        using R = decltype(f(std::declval<T>()));
        return R([action = this->action, f]() {
            T result = action();
            return f(result).run();
        });
    }

    // fmap
    template<typename F>
    auto fmap(F f) const {
        using NewT = decltype(f(std::declval<T>()));
        return IO<NewT>([action = this->action, f]() {
            return f(action());
        });
    }

    // 执行副作用
    T run() const {
        return action();
    }
};

// IO 辅助函数
IO<std::string> get_line() {
    return IO<std::string>([]() {
        std::string line;
        std::getline(std::cin, line);
        return line;
    });
}

IO<int> put_line(const std::string& s) {
    return IO<int>([s]() {
        std::cout << s << "\n";
        return 0;
    });
}

void io_monad_demo() {
    std::cout << "=== IO Monad ===\n";

    // 纯值转换(不执行)
    auto program = IO<int>::pure(42)
        .fmap([](int x) { return x * 2; })
        .fmap([](int x) { return x + 10; });

    std::cout << "Pure computation (42 * 2 + 10): " << program.run() << "\n";

    // 组合 I/O 操作
    std::cout << "Enter a number: ";
    auto input_program = get_line()
        .fmap([](const std::string& s) {
            try {
                return std::stoi(s);
            } catch (...) {
                return 0;
            }
        })
        .fmap([](int x) { return x * 2; })
        .bind([](int x) {
            return put_line("Result: " + std::to_string(x))
                .fmap([x](int) { return x; });
        });

    input_program.run();
    std::cout << "\n";
}
```

**要点**:
- IO 封装副作用,构建阶段不执行
- 只有调用 `run()` 时才真正执行
- 分离纯计算和副作用,易于测试
- 组合多个 IO 操作形成复杂流程

### 5. Monad Laws 验证示例

```cpp
void monad_laws_demo() {
    std::cout << "=== Monad Laws ===\n";

    // Left Identity: return(a).bind(f) == f(a)
    auto f = [](int x) { return Maybe<int>::just(x * 2); };
    int a = 5;

    auto left1 = Maybe<int>::just(a).bind(f);
    auto left2 = f(a);

    std::cout << "Left Identity: "
              << (left1.get() == left2.get() ? "✓" : "✗") << "\n";

    // Right Identity: m.bind(return) == m
    auto m = Maybe<int>::just(10);
    auto right1 = m.bind([](int x) { return Maybe<int>::just(x); });
    auto right2 = m;

    std::cout << "Right Identity: "
              << (right1.get() == right2.get() ? "✓" : "✗") << "\n";

    // Associativity: m.bind(f).bind(g) == m.bind(x => f(x).bind(g))
    auto g = [](int x) { return Maybe<int>::just(x + 10); };

    auto assoc1 = m.bind(f).bind(g);
    auto assoc2 = m.bind([f, g](int x) { return f(x).bind(g); });

    std::cout << "Associativity: "
              << (assoc1.get() == assoc2.get() ? "✓" : "✗") << "\n\n";
}
```

**要点**:
- 左单位元:包装后 bind 等于直接调用
- 右单位元:bind 包装函数等于原值
- 结合律:链式 bind 顺序无关
- 三大定律保证 Monad 行为一致性

### 6. 实际应用:配置验证链

```cpp
struct Config {
    std::string host;
    int port;
    std::string username;
};

Result<Config, std::string> validate_host(const Config& cfg) {
    if (cfg.host.empty()) {
        return Result<Config, std::string>::err("Host cannot be empty");
    }
    return Result<Config, std::string>::ok(cfg);
}

Result<Config, std::string> validate_port(const Config& cfg) {
    if (cfg.port < 1 || cfg.port > 65535) {
        return Result<Config, std::string>::err("Invalid port");
    }
    return Result<Config, std::string>::ok(cfg);
}

Result<Config, std::string> validate_username(const Config& cfg) {
    if (cfg.username.length() < 3) {
        return Result<Config, std::string>::err("Username too short");
    }
    return Result<Config, std::string>::ok(cfg);
}

void validation_pipeline_demo() {
    std::cout << "=== Validation Pipeline ===\n";

    Config valid_cfg{"localhost", 8080, "alice"};
    auto result1 = Result<Config, std::string>::ok(valid_cfg)
        .bind(validate_host)
        .bind(validate_port)
        .bind(validate_username);

    std::cout << "Valid config: "
              << (result1.is_ok() ? "✓" : "✗ " + result1.unwrap_err()) << "\n";

    Config invalid_cfg{"", 8080, "alice"};
    auto result2 = Result<Config, std::string>::ok(invalid_cfg)
        .bind(validate_host)
        .bind(validate_port)
        .bind(validate_username);

    std::cout << "Invalid config: "
              << (result2.is_ok() ? "✓" : "✗ " + result2.unwrap_err()) << "\n\n";
}
```

**要点**:
- Result Monad 串联多个验证步骤
- 遇到第一个失败则短路,返回错误
- 比异常更清晰,错误可预期
- 验证逻辑组合简洁,易扩展

### 运行完整示例

```cpp
int main() {
    maybe_monad_demo();
    result_monad_demo();
    list_monad_demo();
    io_monad_demo();
    monad_laws_demo();
    validation_pipeline_demo();

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
./bin/07_monads
```

**GitHub 完整源码**: [07_monads.cpp](https://github.com/wzxzhuxi/cpp-functional-programming/blob/main/examples/advanced/07_monads.cpp)

### 示例输出

```
=== Maybe Monad ===
100 / 5, then sqrt: 4.47214
100 / 0 (failed): Nothing
42 * 2 + 10 = 94

=== Result Monad ===
77°F = 25°C
Error: Invalid integer: invalid
10 * 2 + 5 = 25

=== List Monad ===
Combinations: 1a 1b 2a 2b 3a 3b
Expanded: 1 10 100 2 20 200 3 30 300

=== IO Monad ===
Pure computation (42 * 2 + 10): 94
Enter a number: 5
Result: 10

=== Monad Laws ===
Left Identity: ✓
Right Identity: ✓
Associativity: ✓

=== Validation Pipeline ===
Valid config: ✓
Invalid config: ✗ Host cannot be empty
```

### 关键要点总结

1. **Monad 是带操作的容器**:封装值并提供 `pure` 和 `bind` 操作
2. **Maybe Monad 处理空值**:避免嵌套 null 检查,链式传递
3. **Result Monad 处理错误**:比异常更显式,错误自动传播
4. **List Monad 处理多值**:flatMap 语义,适合组合和笛卡尔积
5. **IO Monad 隔离副作用**:标记不纯操作,延迟执行
6. **三大定律保证一致性**:左单位元、右单位元、结合律
7. **实际价值在组合**:小函数链式组合,避免嵌套和样板代码
8. **不要过度使用**:简单场景用简单方法,复杂场景才用 Monad

---

## 练习题

本章提供了配套的练习题,帮助你巩固所学知识:

1. 实现 State Monad,支持状态的读取和更新
2. 用 Maybe Monad 重写嵌套的 null 检查代码
3. 实现 Writer Monad,记录计算过程的日志
4. 用 Result Monad 实现配置文件解析器
5. 验证自定义 Monad 满足三大定律
6. 组合多个 Monad 实现复杂业务逻辑

**[查看练习题 →](https://github.com/wzxzhuxi/cpp-functional-programming/blob/main/exercises/07_monads_exercises.cpp)**

练习题包含:
- 详细的问题描述
- 测试用例和预期输出
- 难度分级和提示

### 参考答案

⚠️ **建议先独立完成练习再查看答案**

**[查看参考答案 →](https://github.com/wzxzhuxi/cpp-functional-programming/blob/main/exercises/solutions/07_monads_solutions.cpp)**

答案包含:
- 完整的实现代码
- 详细的解题思路
- 常见错误分析
