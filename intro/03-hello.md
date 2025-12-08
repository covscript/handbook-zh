# 1.3 你好，世界！
在开始之前，我们还需要介绍一下如何编写 CovScript 代码

与大多数编程语言相似，我们首先需要将代码保存在一个文本文件中才能解释或编译

## 1.3.1 文件类型

CovScript 的源代码有如下几种类型：
 + `*.csc`: CovScript 3 源文件
 + `*.csp`: CovScript 3 软件包
 + `*.ecs`: CovScript 4 源文件

其中：
 + `*.csc` 和 `*.csp` 能直接运行，无需编译
 + `*.ecs` 文件在运行时会自动编译成 `*.csc` 格式并缓存，也可以手动编译

**ECS 与 CSC 的关系：** CovScript 4 (ECS) 是 CovScript 3 (CSC) 的超集，提供了更多语言特性。ECS 文件编译后生成 CSC 代码（类似于 TypeScript 编译为 JavaScript），可以在 CSC 解释器上运行

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
# 使用 UTF-8 编码集
@charset: utf8
# 若在 Windows 上可以使用 @charset: gbk

# 注释：使用 # 符号
# 这是一个简单的程序示例

# 定义变量
var name = "Alice"
var age = 25

# 输出信息
system.out.println("姓名：" + name)
system.out.println("年龄：" + to_string(age))

# 简单的计算
var radius = 5
var area = math.constants.pi * (radius ^ 2)
system.out.println("圆的面积：" + to_string(area))
```

## 1.3.4 运行程序的方式

### 直接运行 CSC 文件

对于 `.csc` 文件，可以直接运行：

```bash
cs program.csc
```

**传递命令行参数**：
```bash
cs program.csc arg1 arg2 arg3
```

在程序中可以通过 `context.cmd_args` 访问这些参数。

### 运行 ECS 文件（CovScript 4）

**注意：** 以下功能需要安装 ECS 引导程序（`cspkg install ecs_bootstrap`）。

对于 `.ecs` 文件，可以直接运行，ECS 会自动缓存编译后的文件，只在文件变化时重新编译：

```bash
# 直接运行 ECS 文件（自动编译并缓存）
ecs program.ecs
```

**显式编译**：

```bash
# 编译 ECS 文件为 CSC 格式
ecs program.ecs -o program.csc

# 运行编译后的文件
cs program.csc
```

**编译为包文件**：

```bash
# 编译为 .csp 包文件（适用于 package 声明的文件）
ecs my_package.ecs -o my_package.csp
```

**运行原理：** ECS 编译器会将 `.ecs` 文件编译为标准的 `.csc` 字节码，然后由 CSC 解释器执行。编译后的文件会被缓存到 `.ecs_cache` 目录，只有当源文件修改时才会重新编译。

### 交互式运行（REPL）

CovScript 支持交互式 REPL（Read-Eval-Print Loop）模式，适合快速测试代码片段：

```bash
cs
```

进入 REPL 后，可以直接输入代码并立即看到结果：

```
> var x = 10
> var y = 20
> system.out.println(x + y)
30
> var list = new list
> list.push_back("Hello")
> list.push_back("World")
> foreach item in list
>     system.out.println(item)
> end
Hello
World
> quit()
```

**REPL 特性**：
- 支持多行输入（如函数定义、循环等）
- 保持变量和函数的状态
- 可以导入模块和包
- 使用 `quit()` 或 `Ctrl+C` 退出

## 1.3.5 调试程序

CovScript 提供了专门的调试器 `cs_dbg`，用于调试程序：

```bash
cs_dbg program.csc
```

### 调试器基本命令

```
help          - 显示帮助信息
run           - 运行程序
step          - 单步执行
next          - 执行到下一行
continue      - 继续执行直到下一个断点
break <line>  - 在指定行设置断点
print <var>   - 打印变量的值
list          - 显示当前代码
quit          - 退出调试器
```

### 调试示例

```bash
$ cs_dbg test.csc
(cs_dbg) break 5        # 在第5行设置断点
(cs_dbg) run            # 开始运行
(cs_dbg) print x        # 打印变量 x 的值
(cs_dbg) step           # 单步执行
(cs_dbg) continue       # 继续执行
```

**调试技巧**：
- 在关键代码处设置断点
- 使用 `print` 命令查看变量状态
- 利用单步执行追踪程序流程
- 结合 `system.out.println()` 输出调试信息

## 1.3.6 常见问题

### Q: 中文字符显示乱码怎么办？

**A**: 在文件开头添加字符集声明：

```covscript
@charset: utf8
```

或在 Windows 上使用 GBK：

```covscript
@charset: gbk
```

### Q: 如何指定文件编码？

**A**: CovScript 默认使用 UTF-8 编码。确保你的文本编辑器保存文件时使用 UTF-8 编码。

### Q: 程序运行报错找不到模块怎么办？

**A**: 确保已经安装了所需的扩展包：

```bash
cspkg install <package_name> --yes
```

### Q: ECS 和 CSC 有什么区别？

**A**: 
- **ECS (CovScript 4)**：提供更多语言特性，需要编译为 CSC
- **CSC (CovScript 3)**：直接解释执行，性能稳定
- ECS 是 CSC 的超集，ECS 代码编译后可在 CSC 上运行

### Q: 如何查看 CovScript 的版本信息？

**A**: 
```bash
cs -v        # CovScript 解释器版本
ecs -v       # ECS 编译器版本（如果已安装）
cspkg -v     # 包管理器版本
```

## 1.3.7 更多示例

查看更多示例代码：

- **官方示例集**：https://github.com/covscript/covscript-example
- **测试用例**：https://github.com/covscript/covscript/tree/master/tests
- **本手册的实战案例**：[2.14 实战案例](../syntax/14-examples.md)

## 下一步

恭喜你完成了入门章节！现在可以继续学习：

- [**贰 · 语法基础**](../syntax/README.md) - 深入学习 CovScript 的语法和特性
- [**叁 · 生态系统与扩展库**](../ecosystem/README.md) - 了解网络、数据库、GUI 等扩展功能
