---
title: 环境配置指南
book: cpp-functional-programming
order: 0
chapter_number: 0
section_type: setup
difficulty: 基础
summary: 配置 C++ 开发环境，准备开始函数式编程之旅
permalink: /books/cpp-functional-programming/setup/
estimated_time: "20分钟"
---

## 环境要求

本教程需要支持现代 C++ 特性的开发环境：

### 编译器

至少需要以下任一编译器：

- **GCC 10+** (推荐 GCC 11 或更高)
- **Clang 12+** (推荐 Clang 14 或更高)
- **MSVC 2019+** (Visual Studio 2019 或更高)

**检查编译器版本**:

```bash
# GCC
g++ --version

# Clang
clang++ --version

# MSVC (在 Developer Command Prompt 中)
cl
```

### C++ 标准

- **最低要求**: C++17
- **推荐使用**: C++20

本教程的代码示例主要使用 C++17 特性，部分高级示例会用到 C++20 的新功能（如 Ranges、Concepts）。

### 构建工具

- **CMake 3.15+** (推荐 3.20+)

CMake 是跨平台的构建系统，用于管理项目编译过程。

**检查 CMake 版本**:

```bash
cmake --version
```

**安装 CMake**:

```bash
# Ubuntu/Debian
sudo apt-get install cmake

# macOS (使用 Homebrew)
brew install cmake

# Windows: 从 https://cmake.org/download/ 下载安装包
```

## 获取代码

### 克隆仓库

```bash
git clone https://github.com/wzxzhuxi/cpp-functional-programming.git
cd cpp-functional-programming
```

### 仓库结构

```
cpp-functional-programming/
├── 01-basics/              # 第1章：基础概念
├── 02-immutability/        # 第2章：不可变性
├── 03-pure-functions/      # 第3章：纯函数
├── 04-higher-order-functions/  # 第4章：高阶函数
├── 05-functional-composition/  # 第5章：函数组合
├── 06-algebraic-types/     # 第6章：代数数据类型
├── 07-monads/              # 第7章：Monad 模式
├── 08-advanced-patterns/   # 第8章：高级模式
├── examples/               # 完整示例程序
│   ├── basic/              # 基础示例
│   ├── intermediate/       # 中级示例
│   └── advanced/           # 高级示例
├── exercises/              # 练习题
│   └── solutions/          # 参考答案
├── CMakeLists.txt          # 根构建配置
└── README.md
```

## 构建项目

### 1. 创建构建目录

```bash
mkdir build
cd build
```

### 2. 配置项目

```bash
# 使用默认编译器
cmake ..

# 或指定编译器
cmake -DCMAKE_CXX_COMPILER=g++-11 ..
cmake -DCMAKE_CXX_COMPILER=clang++-14 ..
```

### 3. 编译

```bash
# Linux/macOS
make

# Windows (MSVC)
cmake --build .

# 或使用并行编译加速
make -j$(nproc)  # Linux
make -j$(sysctl -n hw.ncpu)  # macOS
```

### 4. 运行示例

所有可执行文件会生成在 `build/bin/` 目录：

```bash
# 运行第一章示例
./bin/01_lambdas

# 运行第七章示例
./bin/07_monads

# 运行第一章练习
./bin/01_basics_exercises

# 运行第一章答案
./bin/01_basics_solutions
```

## IDE 配置

### Visual Studio Code

**推荐扩展**:
- C/C++ (Microsoft)
- CMake Tools
- clangd (可选，更好的代码补全)

**配置 CMake**:
1. 安装 CMake Tools 扩展
2. 按 `Ctrl+Shift+P` → 输入 "CMake: Configure"
3. 选择编译器
4. 按 `F7` 或点击底部状态栏 "Build" 按钮编译

### CLion

CLion 原生支持 CMake：
1. 打开项目目录
2. CLion 自动检测 CMakeLists.txt
3. 选择工具链 (Settings → Build, Execution, Deployment → Toolchains)
4. 点击 "Build" 或按 `Ctrl+F9`

### Visual Studio 2019/2022

Visual Studio 原生支持 CMake 项目：
1. File → Open → CMake → 选择 CMakeLists.txt
2. 等待 CMake 配置完成
3. 选择配置 (Debug/Release)
4. Build → Build All

### Vim/Neovim

使用 YouCompleteMe 或 clangd LSP：

**生成 compile_commands.json**:
```bash
cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=ON ..
ln -s build/compile_commands.json .
```

## 常见问题

### Q: 编译时提示 C++17 特性不支持

**A**: 检查 CMakeLists.txt 中的 C++ 标准设置：

```cmake
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
```

手动指定标准：

```bash
cmake -DCMAKE_CXX_STANDARD=20 ..
```

### Q: Windows 下找不到编译器

**A**: 使用 "Developer Command Prompt for VS 2019/2022"
- 或在 PowerShell 中运行：
  ```powershell
  & "C:\Program Files\Microsoft Visual Studio\2022\Community\Common7\Tools\Launch-VsDevShell.ps1"
  ```

### Q: macOS 下 Clang 版本过旧

**A**: 使用 Homebrew 安装最新 LLVM：

```bash
brew install llvm
export PATH="/usr/local/opt/llvm/bin:$PATH"
```

### Q: 链接错误 (undefined reference)

**A**: 确保编译了所有依赖文件。使用 CMake 构建系统可以自动处理依赖关系。

## 下一步

环境配置完成后，开始学习：

**→ [第一章：基础概念 - 理论篇](/books/cpp-functional-programming/01-basics/)**

## 获取帮助

遇到问题？
- 查看 [GitHub Issues](https://github.com/wzxzhuxi/cpp-functional-programming/issues)
- 提交新的 Issue 描述你的问题
- 参考项目 README 的故障排除部分
