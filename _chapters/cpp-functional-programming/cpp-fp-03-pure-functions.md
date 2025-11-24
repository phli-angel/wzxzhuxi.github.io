---
title: 第三章：纯函数
book: cpp-functional-programming
order: 3
chapter_number: 3
difficulty: 核心
summary: 什么是纯函数、副作用、引用透明性、副作用隔离
permalink: /books/cpp-functional-programming/03-pure-functions/
estimated_time: "45分钟"
github_exercises: "https://github.com/wzxzhuxi/cpp-functional-programming/blob/main/exercises/03_pure_functions_exercises.cpp"
github_solutions: "https://github.com/wzxzhuxi/cpp-functional-programming/blob/main/exercises/solutions/03_pure_functions_solutions.cpp"
---

## 理论概念

### 什么是纯函数

**纯函数(Pure Function)**：满足两个条件的函数。

1. **相同输入 → 相同输出** - 无论调用多少次,结果始终一致
2. **无副作用** - 不修改外部状态,不依赖外部可变状态

```cpp
// 纯函数
int add(int a, int b) {
    return a + b;
}

// 不纯：依赖外部可变状态
int counter = 0;
int bad_add(int a, int b) {
    return a + b + counter;  // 结果依赖 counter
}

// 不纯：修改外部状态
int worse_add(int a, int b) {
    counter++;  // 副作用！
    return a + b;
}

// 不纯：I/O 操作
int terrible_add(int a, int b) {
    std::cout << a + b;  // 副作用：输出
    return a + b;
}
```

### 副作用(Side Effects)

副作用是指函数执行时对外部世界的任何可观察变化：

- 修改全局变量或静态变量
- 修改函数参数(非 const 引用)
- I/O 操作(打印、文件读写、网络)
- 抛出异常
- 调用有副作用的函数
- 修改数据库状态
- 获取当前时间、随机数

**纯函数不产生副作用。**

#### 副作用示例对比

```cpp
// 有副作用：修改参数
void bad_normalize(std::vector<int>& vec) {
    int sum = std::accumulate(vec.begin(), vec.end(), 0);
    for (auto& x : vec) x /= sum;
}

// 纯函数：返回新值
std::vector<int> normalize(const std::vector<int>& vec) {
    int sum = std::accumulate(vec.begin(), vec.end(), 0);
    std::vector<int> result;
    result.reserve(vec.size());
    for (int x : vec) {
        result.push_back(x / sum);
    }
    return result;
}

// 有副作用：依赖和修改全局状态
int request_id = 0;
int get_next_id() {
    return ++request_id;  // 不纯！
}

// 纯函数：明确传递状态
int next_id(int current) {
    return current + 1;
}
```

### 引用透明性(Referential Transparency)

**引用透明**：表达式可以被其求值结果替换,而不改变程序行为。

纯函数具有引用透明性。

```cpp
// 纯函数
int square(int x) { return x * x; }

// 引用透明：可以直接替换
int a = square(5);                // 25
int b = square(5) + square(5);    // 50

// 等价于
int a = 25;
int b = 25 + 25;  // 编译器可以优化

// 不纯函数
int global = 10;
int bad_square(int x) {
    return x * x + global;  // 依赖外部状态
}

// 非引用透明：不能直接替换
int c = bad_square(5);  // 结果依赖 global 当前值
global = 20;
int d = bad_square(5);  // 结果改变！
```

引用透明性的好处：
- **编译器优化** - 可以缓存、重排、并行化
- **易于推理** - 函数调用可以直接看作值
- **可缓存** - 相同输入的结果可以记住(memoization)

### 纯函数的好处

#### 1. 易于测试

```cpp
// 纯函数：测试简单直接
int multiply(int a, int b) {
    return a * b;
}

// 测试：不需要 setup/teardown
assert(multiply(2, 3) == 6);
assert(multiply(0, 100) == 0);
assert(multiply(-2, 3) == -6);

// 不纯函数：测试困难
class Database {
    int query_count = 0;
public:
    int query(const std::string& sql) {
        query_count++;  // 副作用
        // ... 数据库操作
        return 42;
    }
};

// 测试需要复杂的 mock/stub
```

#### 2. 易于并行

```cpp
// 纯函数：天然线程安全
std::vector<int> data = {1, 2, 3, 4, 5};
std::vector<int> result(data.size());

// 可以安全并行
std::transform(std::execution::par,
               data.begin(), data.end(), result.begin(),
               [](int x) { return x * x; });  // 纯函数

// 不纯函数：数据竞争
int sum = 0;
std::for_each(data.begin(), data.end(),
              [&sum](int x) { sum += x; });  // 危险！多线程会出错
```

#### 3. 易于组合

```cpp
// 纯函数可以随意组合
auto f = [](int x) { return x * 2; };
auto g = [](int x) { return x + 1; };
auto h = [](int x) { return g(f(x)); };  // 组合：先 f 后 g

std::vector<int> nums = {1, 2, 3};
auto result = nums
    | std::views::transform(f)
    | std::views::transform(g);
```

#### 4. 易于推理和调试

```cpp
// 纯函数：出错就是这个函数的问题
int calculate(int x, int y) {
    return x * 2 + y * 3;
}

// 不纯：问题可能在任何地方
extern int secret_global;
int bad_calculate(int x, int y) {
    secret_global += x;
    return x * 2 + y * 3 + secret_global;
}
// 出错了？可能是其他地方改了 secret_global
```

### 副作用隔离

实际程序不可能完全没有副作用(否则没有输出)。关键是**隔离副作用**。

#### 核心原则

1. **副作用推到边界** - 核心逻辑纯函数,I/O 在外层
2. **明确标记** - 有副作用的函数清晰命名或注释
3. **最小化范围** - 副作用的代码越少越好

#### 示例：隔离 I/O

```cpp
// 差：逻辑和 I/O 混在一起
void process_and_print() {
    auto data = read_file("input.txt");  // I/O
    auto result = transform(data);
    write_file("output.txt", result);   // I/O
}

// 好：分离逻辑和 I/O
// 纯函数：核心逻辑
std::vector<int> transform(const std::vector<int>& data) {
    std::vector<int> result;
    for (int x : data) {
        result.push_back(x * 2);
    }
    return result;
}

// 不纯：只负责 I/O
void run() {
    auto data = read_file("input.txt");     // 副作用
    auto result = transform(data);          // 纯函数
    write_file("output.txt", result);       // 副作用
}
```

#### 示例：隔离状态

```cpp
// 差：状态和逻辑耦合
class BadCounter {
    int count = 0;
public:
    void process(int x) {
        count += x;  // 修改状态
        // ... 其他逻辑
    }
    int get() const { return count; }
};

// 好：状态分离
// 纯函数
int add(int current, int delta) {
    return current + delta;
}

// 状态管理
class Counter {
    int count = 0;
public:
    void update(int x) {
        count = add(count, x);  // 调用纯函数
    }
    int get() const { return count; }
};
```

### 不纯函数的合理使用

有些场景必须不纯,但可以最小化影响：

#### 1. 日志和调试

```cpp
// 纯函数主逻辑
int complex_calculation(int x, int y) {
    int step1 = x * 2;
    int step2 = y + 10;
    int result = step1 + step2;

    // 副作用：调试日志(可以移除)
    #ifdef DEBUG
    std::cerr << "Debug: " << result << "\n";
    #endif

    return result;
}
```

#### 2. 性能优化(缓存)

```cpp
// 纯函数,但内部缓存(对外透明)
int expensive_calculation(int n) {
    static std::unordered_map<int, int> cache;

    auto it = cache.find(n);
    if (it != cache.end()) {
        return it->second;  // 缓存命中
    }

    // 计算
    int result = /* ... */;
    cache[n] = result;  // 副作用：修改缓存
    return result;
}
// 对外仍是纯函数：相同输入 → 相同输出
```

#### 3. I/O 必须在某处发生

```cpp
// 架构：纯函数核心 + 不纯外壳
int main() {
    // 不纯：读取输入
    auto input = read_input();

    // 纯函数：所有业务逻辑
    auto result = process(input);

    // 不纯：输出结果
    write_output(result);

    return 0;
}
```

### const 与纯函数

C++ 用 `const` 帮助实现纯函数：

```cpp
// const 成员函数：承诺不修改对象
class Point {
    int x_, y_;
public:
    // 纯：只读
    int x() const { return x_; }
    int distance_from_origin() const {
        return std::sqrt(x_ * x_ + y_ * y_);
    }

    // 不纯：修改状态
    void move(int dx, int dy) {
        x_ += dx;
        y_ += dy;
    }
};

// const 参数：承诺不修改输入
int sum(const std::vector<int>& nums) {
    return std::accumulate(nums.begin(), nums.end(), 0);
}
```

### 实践建议

1. **默认写纯函数** - 除非必须有副作用
2. **用 const** - 参数和成员函数都加 `const`
3. **返回新值** - 不修改输入,返回计算结果
4. **副作用集中** - I/O、状态修改在明确的地方
5. **命名清晰** - `get_*` 应该是纯的,`set_*`/`update_*` 可以不纯

### 关键要点总结

1. **纯函数 = 可预测** - 相同输入永远相同输出
2. **无副作用 = 安全** - 不改变外部世界
3. **引用透明 = 可优化** - 编译器可以自由优化
4. **易于测试** - 不需要 mock,直接断言
5. **易于并行** - 天然线程安全

---

## 代码示例

### 代码示例概览

本节包含纯函数的完整可运行示例,涵盖：

1. **纯函数 vs 不纯函数** - 对比示例
2. **副作用识别** - 常见副作用类型
3. **副作用隔离** - 架构设计模式
4. **引用透明性** - 优化案例
5. **实际应用** - 纯函数式数据处理

### 示例 1：纯函数 vs 不纯函数

```cpp
// 纯函数：相同输入 → 相同输出
int pure_add(int a, int b) {
    return a + b;
}

// 不纯：依赖全局状态
int global_counter = 0;
int impure_add_read(int a, int b) {
    return a + b + global_counter;  // 结果不可预测
}

// 不纯：修改全局状态
int impure_add_write(int a, int b) {
    global_counter++;  // 副作用
    return a + b;
}

// 不纯：I/O 操作
int impure_add_io(int a, int b) {
    std::cout << "Adding " << a << " and " << b << "\n";  // 副作用
    return a + b;
}

void example_pure_vs_impure() {
    // 纯函数：可预测
    assert(pure_add(2, 3) == 5);
    assert(pure_add(2, 3) == 5);  // 永远是 5

    // 不纯：结果依赖状态
    global_counter = 10;
    assert(impure_add_read(2, 3) == 15);
    global_counter = 20;
    assert(impure_add_read(2, 3) == 25);  // 结果改变了！
}
```

**要点**:
- 纯函数结果永远一致
- 不纯函数结果依赖外部状态
- 纯函数更易测试和调试

### 示例 2：修改参数的副作用

```cpp
// 不纯：修改输入参数
void bad_double_values(std::vector<int>& vec) {
    for (auto& x : vec) {
        x *= 2;  // 修改了输入！
    }
}

// 纯函数：返回新向量
std::vector<int> good_double_values(const std::vector<int>& vec) {
    std::vector<int> result;
    result.reserve(vec.size());
    for (int x : vec) {
        result.push_back(x * 2);
    }
    return result;
}

void example_parameter_mutation() {
    std::vector<int> nums = {1, 2, 3, 4, 5};

    // 使用不纯版本
    auto nums_copy1 = nums;
    bad_double_values(nums_copy1);
    // nums_copy1 被修改了：{2, 4, 6, 8, 10}

    // 使用纯函数版本
    auto doubled = good_double_values(nums);
    // nums 不变：{1, 2, 3, 4, 5}
    // doubled 是新向量：{2, 4, 6, 8, 10}
}
```

### 示例 3：隔离 I/O 副作用

```cpp
// 纯函数：数据处理逻辑
std::vector<int> process_data(const std::vector<int>& input) {
    std::vector<int> result;
    for (int x : input) {
        if (x % 2 == 0) {
            result.push_back(x * x);
        }
    }
    return result;
}

// 不纯：I/O 操作
std::vector<int> read_numbers(const std::string& filename) {
    std::ifstream file(filename);
    std::vector<int> numbers;
    int num;
    while (file >> num) {
        numbers.push_back(num);
    }
    return numbers;
}

void write_numbers(const std::string& filename, const std::vector<int>& numbers) {
    std::ofstream file(filename);
    for (int num : numbers) {
        file << num << "\n";
    }
}

// 架构：纯函数核心,I/O 在边界
void run_pipeline(const std::string& input_file, const std::string& output_file) {
    // 副作用：读取
    auto input = read_numbers(input_file);

    // 纯函数：处理
    auto output = process_data(input);

    // 副作用：写入
    write_numbers(output_file, output);
}
```

**要点**:
- 核心逻辑(`process_data`)是纯函数
- I/O 操作隔离在外层
- 纯函数部分易于测试

### 示例 4：引用透明性与优化

```cpp
// 纯函数：引用透明
int expensive_calculation(int x) {
    // 模拟耗时计算
    int result = 0;
    for (int i = 0; i < x * 1000; ++i) {
        result += i % 10;
    }
    return result;
}

// 优化：memoization(缓存结果)
int memoized_calculation(int x) {
    static std::unordered_map<int, int> cache;

    auto it = cache.find(x);
    if (it != cache.end()) {
        return it->second;  // 缓存命中
    }

    int result = expensive_calculation(x);
    cache[x] = result;
    return result;
}

void example_referential_transparency() {
    // 因为 expensive_calculation 是纯函数,
    // 可以安全地缓存结果

    auto start1 = std::chrono::high_resolution_clock::now();
    int r1 = memoized_calculation(100);
    auto end1 = std::chrono::high_resolution_clock::now();

    auto start2 = std::chrono::high_resolution_clock::now();
    int r2 = memoized_calculation(100);  // 第二次：从缓存
    auto end2 = std::chrono::high_resolution_clock::now();

    // 第二次调用几乎瞬间完成
    auto duration1 = std::chrono::duration_cast<std::chrono::microseconds>(end1 - start1);
    auto duration2 = std::chrono::duration_cast<std::chrono::microseconds>(end2 - start2);

    std::cout << "First call:  " << duration1.count() << " μs\n";
    std::cout << "Second call: " << duration2.count() << " μs\n";
}
```

### 示例 5：纯函数的数学计算库

```cpp
namespace Math {
    // 所有函数都是纯函数

    double square(double x) {
        return x * x;
    }

    double cube(double x) {
        return x * x * x;
    }

    double distance(double x1, double y1, double x2, double y2) {
        return std::sqrt(square(x2 - x1) + square(y2 - y1));
    }

    std::vector<double> normalize(const std::vector<double>& vec) {
        double sum = std::accumulate(vec.begin(), vec.end(), 0.0);
        std::vector<double> result;
        result.reserve(vec.size());
        for (double x : vec) {
            result.push_back(x / sum);
        }
        return result;
    }

    // 组合纯函数
    double euclidean_norm(const std::vector<double>& vec) {
        double sum_squares = 0.0;
        for (double x : vec) {
            sum_squares += square(x);
        }
        return std::sqrt(sum_squares);
    }
}

void example_pure_math_library() {
    assert(Math::square(5) == 25);
    assert(Math::cube(3) == 27);
    assert(Math::distance(0, 0, 3, 4) == 5.0);

    std::vector<double> data = {1, 2, 3, 4};
    auto normalized = Math::normalize(data);
    // normalized: {0.1, 0.2, 0.3, 0.4}
}
```

### 示例 6：状态隔离

```cpp
// 纯函数：状态转换
struct GameState {
    int score;
    int level;
    bool game_over;
};

GameState update_score(const GameState& state, int points) {
    return GameState{state.score + points, state.level, state.game_over};
}

GameState level_up(const GameState& state) {
    return GameState{state.score, state.level + 1, state.game_over};
}

GameState end_game(const GameState& state) {
    return GameState{state.score, state.level, true};
}

// 不纯：管理状态
class Game {
    GameState state_ = {0, 1, false};

public:
    void add_score(int points) {
        state_ = update_score(state_, points);  // 调用纯函数
    }

    void advance_level() {
        state_ = level_up(state_);
    }

    void game_over() {
        state_ = end_game(state_);
    }

    const GameState& get_state() const { return state_; }
};

void example_state_isolation() {
    // 纯函数测试：简单
    auto s1 = GameState{100, 1, false};
    auto s2 = update_score(s1, 50);
    assert(s2.score == 150);
    assert(s1.score == 100);  // s1 未改变

    // 状态管理：只在 Game 类中
    Game game;
    game.add_score(100);
    game.advance_level();
    assert(game.get_state().score == 100);
    assert(game.get_state().level == 2);
}
```

### 示例 7：并行安全的纯函数

```cpp
// 纯函数：天然线程安全
int expensive_transform(int x) {
    int result = x;
    for (int i = 0; i < 1000; ++i) {
        result = (result * 1103515245 + 12345) % 2147483648;
    }
    return result % 100;
}

void example_parallel_pure_functions() {
    std::vector<int> data(1000000);
    std::iota(data.begin(), data.end(), 0);

    std::vector<int> result(data.size());

    // 并行执行：安全,因为 expensive_transform 是纯函数
    std::transform(std::execution::par,
                   data.begin(), data.end(),
                   result.begin(),
                   expensive_transform);

    // 不需要锁,不需要担心数据竞争
}
```

### 运行完整示例

下载并运行：

```bash
git clone https://github.com/wzxzhuxi/cpp-functional-programming.git
cd cpp-functional-programming
mkdir build && cd build
cmake ..
make
./bin/03_pure_functions
```

**完整源码**: [03_pure_functions.cpp](https://github.com/wzxzhuxi/cpp-functional-programming/blob/main/examples/basic/03_pure_functions.cpp)

### 关键要点总结

1. **纯函数 = 可预测** - 测试简单,调试容易
2. **const 是关键** - 参数和方法都用 const
3. **副作用隔离** - 核心逻辑纯函数,I/O 在边界
4. **引用透明** - 可以安全缓存结果
5. **并行友好** - 天然线程安全

---

## 练习与答案

本章提供了配套的练习题和参考答案,帮助你巩固所学知识。

### 练习题

在线查看练习题和测试用例：

**[查看练习题 →](https://github.com/wzxzhuxi/cpp-functional-programming/blob/main/exercises/03_pure_functions_exercises.cpp)**

练习包含：
- 详细的问题描述
- 测试用例和预期输出
- 难度分级和提示

### 参考答案

警告 **建议先独立完成练习再查看答案**

**[查看参考答案 →](https://github.com/wzxzhuxi/cpp-functional-programming/blob/main/exercises/solutions/03_pure_functions_solutions.cpp)**

答案包含：
- 完整的实现代码
- 详细的解题思路
- 常见错误分析
