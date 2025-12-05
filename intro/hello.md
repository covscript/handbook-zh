# 1.3 你好，世界！
在开始之前，我们还需要介绍一下如何编写 CovScript 代码

与大多数编程语言相似，我们首先需要将代码保存在一个文本文件中才能解释或编译

## 1.3.1 文件类型

CovScript 的源代码有如下几种类型：
 + `*.csc`: CovScript 3 源文件
 + `*.csp`: CovScript 3 软件包
 + `*.ecs`: CovScript 4 源文件

其中，`*.csc` 和 `*.csp` 能直接运行，无需编译；`*.ecs` 需要编译成 `*.csc` 或 `*.csp` 后才能运行

## 1.3.2 第一个程序

让我们创建第一个 CovScript 程序。使用任何文本编辑器创建一个名为 `hello.csc` 的文件，并输入以下内容：

```covscript
# hello.csc - 我的第一个 CovScript 程序
system.out.println("Hello, World!")
```

保存文件后，在命令行中运行：

```bash
cs hello.csc
```

你应该会看到输出：

```
Hello, World!
```

恭喜！你已经成功运行了第一个 CovScript 程序。

## 1.3.3 程序结构

一个简单的 CovScript 程序通常包含以下内容：

```covscript
# 注释：使用 # 符号
# 这是一个简单的程序示例

# 导入需要的模块（可选）
import math

# 定义变量
var name = "Alice"
var age = 25

# 输出信息
system.out.println("姓名：" + name)
system.out.println("年龄：" + to_string(age))

# 简单的计算
var radius = 5
var area = math.pi * radius * radius
system.out.println("圆的面积：" + to_string(area))
```

## 1.3.4 运行程序的方式

### 直接运行

对于 `.csc` 和 `.csp` 文件，可以直接运行：

```bash
cs program.csc
```

### 编译并运行 ECS 文件

对于 `.ecs` 文件，需要先编译：

```bash
# 编译 ECS 文件
ecs program.ecs -o program.csc

# 运行编译后的文件
cs program.csc
```

### 交互式运行

CovScript 还支持交互式 REPL 模式：

```bash
cs
```

进入 REPL 后，可以直接输入代码并立即看到结果：

```
> var x = 10
> var y = 20
> system.out.println(x + y)
30
> exit()
```

## 1.3.5 调试程序

CovScript 提供了调试器 `cs_dbg`：

```bash
cs_dbg program.csc
```

调试器支持设置断点、单步执行、查看变量等功能。

## 下一步

现在你已经了解了如何编写和运行 CovScript 程序，接下来我们将在 [**贰 · 语法基础**](../syntax/README.md) 章节中深入学习 CovScript 的语法和特性。
