# Github Actions 基础

GitHub 的 **GitHub Actions** 是 GitHub 提供的一套 **CI/CD（持续集成 / 持续部署）自动化平台**。简单来说，它可以在代码仓库发生某些事件时（比如 `push`、`pull_request`），自动运行脚本，例如：

- 自动编译代码
- 自动运行测试
- 自动打包发布
- 自动部署服务器
- 自动格式检查 / 静态分析

对于现在做的 **C++ 项目（threadpool、ini 库等）**，GitHub Actions 非常常见的用途就是：**自动构建 + 自动运行 CTest**。

------

# 一、GitHub Actions 的基本概念

GitHub Actions 主要由几个核心概念组成：

| 概念         | 说明                     |
| ------------ | ------------------------ |
| **Workflow** | 自动化流程               |
| **Event**    | 触发事件（push / PR 等） |
| **Job**      | 一组任务                 |
| **Step**     | Job 里的具体步骤         |
| **Runner**   | 运行任务的机器           |

结构层级是：

```
Workflow
 └── Job
      └── Step
```

# 二、最简单的 Workflow 示例

GitHub Actions 配置文件放在：

```
.github/workflows/
```

例如：

```
.github/workflows/ci.yml
```

最简单例子：

```
name: C++ CI

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Build
        run: |
          g++ main.cpp -o app

      - name: Run
        run: ./app
```

流程：

```
push / PR
   ↓
启动 workflow
   ↓
创建 ubuntu runner
   ↓
checkout 代码
   ↓
编译
   ↓
运行程序
```

# 三、C++ 项目常见 CI 示例

像你现在的 **CMake + CTest 项目**，一般 workflow 是这样的：

```
name: Build & Test

on:
  push:
  pull_request:

jobs:
  ci:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Configure
        run: cmake -B build -S .

      - name: Build
        run: cmake --build build

      - name: Test
        run: ctest --test-dir build --output-on-failure
```

运行流程：

```
push
 ↓
cmake configure
 ↓
cmake build
 ↓
ctest
```

如果 **测试失败**

GitHub PR 页面会显示：

```
❌ CI failed
```

如果成功：

```
✅ All checks passed
```

这就是你之前看到的 **PR status check**。

------

# 四、Matrix（多平台构建）

GitHub Actions 很强的一点是 **矩阵构建**：

```
strategy:
  matrix:
    os: [ubuntu-latest, macos-latest, windows-latest]
    build_type: [Debug, Release]
```

效果：

```
Ubuntu + Debug
Ubuntu + Release
Mac + Debug
Mac + Release
Windows + Debug
Windows + Release
```

一次 PR 自动测试 **6 种环境**。

这也是很多开源 C++ 项目的标准 CI。

------

# 五、常见官方 Action

常用的 Action：

| Action                    | 作用         |
| ------------------------- | ------------ |
| `actions/checkout`        | 拉取仓库代码 |
| `actions/setup-python`    | 安装 Python  |
| `actions/setup-node`      | 安装 Node    |
| `actions/cache`           | 缓存依赖     |
| `actions/upload-artifact` | 上传构建产物 |

例如：

```
- uses: actions/checkout@v4
```

意思是使用官方 action 来 checkout 代码。

# 六、CI / CD 能做什么

GitHub Actions 可以自动化很多事情：

### CI

- 编译代码
- 单元测试
- 静态检查
- clang-format
- coverage

### CD

- 发布 release
- 自动生成文档
- 自动发布 package
- 自动 deploy server

例如：

```
git tag v1.0
   ↓
自动 build
   ↓
自动打包
   ↓
自动发布 GitHub Release
```