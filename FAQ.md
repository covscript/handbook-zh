# CovScript 常见问题解答（FAQ）

本文档汇总了 CovScript 使用过程中的常见问题和解决方案。

## 目录

- [安装与配置](#安装与配置)
- [语法与特性](#语法与特性)
- [标准库](#标准库)
- [扩展库](#扩展库)
- [性能与优化](#性能与优化)
- [开发工具](#开发工具)
- [疑难杂症](#疑难杂症)

---

## 安装与配置

### Q: 如何检查 CovScript 是否安装成功？

**A**: 在命令行中运行以下命令：

```bash
cs -v        # 查看 CovScript 版本
ecs -v       # 查看 ECS 版本（如果已安装）
cspkg -v     # 查看包管理器版本
```

如果显示版本信息，说明安装成功。

### Q: Windows 上安装后命令不可用怎么办？

**A**: 
1. 重启命令行窗口或重启计算机
2. 检查环境变量 `PATH` 是否包含 CovScript 的 bin 目录
3. 手动添加到环境变量：`C:\Program Files\CovScript\bin`

### Q: macOS 上如何配置全局命令？

**A**: 在 `~/.zshrc` 或 `~/.bash_profile` 中添加：

```bash
export CS_DARWIN_PATH="/Applications/CovScript.app/Contents/MacOS/covscript"
export PATH="$PATH:$CS_DARWIN_PATH/bin"
export CS_DEV_PATH="$CS_DARWIN_PATH"
```

然后运行 `source ~/.zshrc` 使配置生效。

### Q: Linux 上依赖问题怎么解决？

**A**: 
```bash
# Ubuntu/Debian
sudo apt-get install -f

# 或手动安装依赖
sudo apt-get install libc libstdc++
```

### Q: 如何更新 CovScript 到最新版本？

**A**: 
1. 下载最新的安装包
2. 运行安装程序（会自动覆盖旧版本）
3. 更新已安装的包：`cspkg update --all`

---

## 语法与特性

### Q: CovScript 是静态类型还是动态类型？

**A**: CovScript 是**动态类型**语言。变量的类型在运行时确定，可以随时改变：

```covscript
var x = 10       # x 是数字
x = "hello"      # 现在 x 是字符串
x = {1, 2, 3}    # 现在 x 是数组
```

### Q: 如何定义多行字符串？

**A**: 使用转义字符 `\n`：

```covscript
var multiline = "第一行\n第二行\n第三行"
system.out.println(multiline)
```

CovScript 不支持三引号多行字符串。

### Q: 数组和列表有什么区别？

**A**: 两者都是动态大小的容器，均提供 `push_back`、`pop_back` 等方法，但内部实现和性能特性不同：

- **数组 (Array)**：`{1, 2, 3}`
  - 底层实现：双端队列
  - 索引方式：支持 `arr[index]` 下标随机访问，O(1) 复杂度
  - 性能特点：头尾插入/删除为 O(1)，中间插入/删除为 O(N)
  - 适用场景：需要随机访问、头尾操作频繁

- **列表 (List)**：`new list`
  - 底层实现：双向链表
  - 索引方式：仅支持迭代器顺序访问，不支持下标
  - 性能特点：已知位置插入/删除为 O(1)，但定位到指定位置需 O(N) 遍历
  - 适用场景：频繁在已知位置插入/删除、不需要随机访问

根据实际业务需求选择合适的数据结构。

### Q: 如何实现类似 Python 的列表推导式？

**A**: CovScript 没有内置列表推导式语法，但可通过循环实现相同功能：

```covscript
# 方法 1：使用传统 for 循环
var squares = new list
for i = 1, i <= 10, ++i
    squares.push_back(i * i)
end
```

```
# 方法 2：使用 foreach 配合 range（更简洁）
var squares = new list
foreach i in range(1, 11)
    squares.push_back(i * i)
end
```

对于带条件筛选的场景：

```covscript
# 生成 1 到 20 中的偶数平方
var even_squares = new list
foreach i in range(1, 21)
    if i % 2 == 0
        even_squares.push_back(i * i)
    end
end
```

### Q: 函数参数是值传递还是引用传递？

**A**: CovScript 中所有参数都是**引用传递**，包括重新赋值也会修改原变量。

```covscript
function modifyArray(arr)
    arr.push_back(100)  # 会修改原数组
end

function reassignNumber(num)
    num = 100  # 重新赋值也会修改原变量
end

var myArr = {1, 2, 3}
modifyArray(myArr)
system.out.println(myArr)  # 输出: {1, 2, 3, 100}

var myNum = 42
reassignNumber(myNum)
system.out.println(myNum)  # 输出: 100（已改变）
```

**安全实践**：如果需要修改参数而不影响原值，必须使用 `clone()` 创建副本：

```covscript
function safeModify(arr)
    var _arr = clone(arr)  # 创建副本，避免修改原数据
    _arr.push_back(100)
    return _arr
end

var original = {1, 2, 3}
var modified = safeModify(original)
system.out.println(original)  # 输出: {1, 2, 3}（未改变）
system.out.println(modified)  # 输出: {1, 2, 3, 100}
```

**常见陷阱**：传递字面量（如 `1`、`"str"`、`true` 等）时要特别小心，因为无法修改字面量：

```covscript
function tryModify(val)
    val = 200  # 错误！不能修改字面量
end

tryModify(100)  # 运行时错误
```

**ECS 特性**：在 ECS 中，`=` 表示**按值捕获**参数，创建副本而非引用：

```covscript
function safeModify(=arr)
    arr.push_back(100)  # 修改的是副本，不影响原数据
end

var original = {1, 2, 3}
safeModify(original)
system.out.println(original)  # 输出: {1, 2, 3}（未改变）
```

### Q: 如何实现函数重载？

**A**: CovScript 不支持函数重载。可以使用可变参数或类型检查：

```covscript
function process(...args)
    if args.size == 1
        # 单参数处理
        return args[0] * 2
    else
        # 多参数处理
        var sum = 0
        foreach arg in args
            sum += arg
        end
        return sum
    end
end
```

### Q: 类的构造函数如何传递参数？

**A**: CSC 和 ECS 在构造函数参数传递上有显著差异：

**CSC**：
- `new` 和 `gcnew` 不支持传参
- 需要手动调用初始化方法

```covscript
class Person
    var name = ""
    var age = 0
    
    function init(n, a)
        this.name = n
        this.age = a
        return this
    end
end

# 方式 1：手动调用初始化方法
var p1 = (new Person).init("Alice", 25)

# 方式 2：工厂函数（推荐）
function createPerson(name, age)
    return (new Person).init(name, age)
end

var p2 = createPerson("Bob", 30)
```

**ECS**：
- `new` 和 `gcnew` 直接支持传参
- 参数自动传递给 `construct` 函数（如果已定义）

```covscript
class Person
    var name = ""
    var age = 0
    
    function construct(n, a)
        this.name = n
        this.age = a
    end
end

var p = new Person{"Alice", 25}
system.out.println(p.name)  # 输出: Alice
```

**最佳实践**：推荐统一使用工厂函数模式，原因如下：
- **跨版本兼容**：同一套代码可在 CSC 和 ECS 中运行
- **语义清晰**：函数名明确表达创建意图（如 `createUser`、`buildConfig`）
- **灵活扩展**：便于添加参数验证、默认值处理、对象池等高级功能
- **便于测试**：工厂函数可轻松模拟（mock）对象创建过程

---

## 标准库

### Q: 如何读取整个文件的内容？

**A**: 使用循环逐行读取：

```covscript
function readFile(filename)
    var file = iostream.ifstream(filename)
    var content = new string

    # 逐行读取直到文件末尾
    loop
        var line = file.getline()
        if !file.good()
            break
        end
        content += line + "\n"
    end

    return content
end

var text = readFile("example.txt")
```

### Q: 如何获取命令行参数？

**A**: 使用 `context.cmd_args`：

```covscript
# 在主程序中
if context.cmd_args.size > 1
    var arg1 = context.cmd_args[1]
    system.out.println("第一个参数: " + arg1)
end
```

或使用 `argparse` 扩展库进行更复杂的参数解析。

### Q: 如何生成随机数？

**A**: 使用 `math` 库的随机函数：

```covscript
# 随机整数（1-100）
var num = math.randint(1, 100)

# 随机浮点数（0.0-1.0）
var f = math.rand()

# 随机浮点数（指定范围）
var f2 = math.rand(10.5, 20.5)
```

### Q: 如何遍历目录中的文件？

**A**: 使用 `system.path.scan()`：

```covscript
var files = system.path.scan(".")
foreach file in files
    if file.type != system.path.type.dir
        system.out.println("文件: " + file.name)
    end
end
```

### Q: 字符串的 `size` 是属性还是方法？

**A**: `size` 是**属性**，不是方法：

```covscript
var str = "hello"
var len = str.size       # 正确
# var len = str.size()   # 错误
```

### Q: 如何实现延时执行？

**A**: 使用 `runtime.delay()`（单位：毫秒）：

```covscript
system.out.println("开始")
runtime.delay(2000)  # 延时 2 秒
system.out.println("结束")
```

---

## 扩展库

### Q: 如何安装扩展库？

**A**: 使用 `cspkg` 包管理器：

```bash
cspkg install <package_name> --yes
```

常用扩展库：
- `network` - 网络编程
- `csdbc_sqlite` - SQLite 数据库
- `imgui` - GUI 编程
- `argparse` - 命令行参数解析

### Q: 导入扩展库时报错怎么办？

**A**: 
1. 确认已安装：`cspkg list`
2. 重新安装：`cspkg install <package_name> --yes`
3. 检查导入路径：`runtime.get_import_path()`

### Q: 如何在异步网络编程中处理多个连接？

**A**: 参考 [3.1 网络编程](./ecosystem/01-network.md) 的服务器示例。

### Q: 数据库操作如何使用预处理语句？

**A**: 

```covscript
import csdbc_sqlite

var db = csdbc_sqlite.connect("test.db")
var stmt = db.prepare("INSERT INTO users (name, age) VALUES (?, ?)")
stmt.bind(1, "Alice")
stmt.bind(2, 25)
stmt.just_exec()
```

详见 [3.2 数据库操作](./ecosystem/02-database.md)。

### Q: ImGui 如何显示中文？

**A**: 设置 UTF-8 模式，并使用 `imgui_font.source_han_sans` 字体：

```covscript
@charset: utf8
import imgui, imgui_font
using imgui

var app = window_application(800, 640, "Test ImGUI Application")
var font = add_font_extend_cn(imgui_font.source_han_sans, 16)
var window_opened = true

while !app.is_closed()
	app.prepare()
	push_font(font)
    begin_window("测试窗口", window_opened, {})
        if !window_opened
            break
        end
        set_window_size(vec2(400, 300))
        # GUI 代码
        text("你好，世界！")
    end_window()
    pop_font()
	app.render()
end
```

---

## 性能与优化

### Q: CovScript 的性能如何？

**A**: CovScript 是解释型语言，性能与 Python、Lua 等相近。对于 CPU 密集型任务，建议：
1. 使用 C++ 编写扩展模块
2. 减少不必要的对象创建
3. 使用合适的数据结构

### Q: 如何提高代码执行速度？

**A**: 
1. **避免频繁的类型转换**
2. **减少字符串拼接**（使用列表收集后一次性拼接）
3. **缓存计算结果**
4. **使用局部变量**而非全局变量
5. **选择合适的数据结构**

### Q: 内存占用太高怎么办？

**A**: CovScript 使用 RAII + 引用 GC 的内存管理方式，理论上不会出现异常内存占用。若遇到问题，可尝试：

1. 及时关闭文件和资源
2. 清空不再使用的大型列表/数组
3. 避免循环引用
4. 使用 `--stack-resize` 调整栈大小（若是栈溢出）

---

## 开发工具

### Q: 有没有 CovScript 的 IDE？

**A**: 推荐使用 **VSCode** + CovScript 扩展：
1. 安装 VSCode
2. 搜索并安装 "CovScript" 扩展（by mikecovlee）
3. 享受语法高亮和代码补全

### Q: 如何调试 CovScript 程序？

**A**: 使用 `cs_dbg` 调试器。

**启动方式**：

```bash
cs_dbg program.csc
```

**启动选项**：

| 选项 | 简称 | 功能 |
|------|------|------|
| `--help` | `-h` | 显示帮助信息 |
| `--version` | `-v` | 显示版本信息 |
| `--silent` | `-s` | 关闭命令提示符 |
| `--wait-before-exit` | `-w` | 进程退出前等待用户输入 |
| `--csym <FILE>` | `-g <FILE>` | 从文件读取 cSYM 符号信息 |
| `--stack-resize <SIZE>` | `-S <SIZE>` | 重设运行时栈大小 |
| `--log-path <PATH>` | `-l <PATH>` | 设置日志输出路径 |
| `--import-path <PATH>` | `-i <PATH>` | 设置模块导入路径 |

**常用调试命令**：

| 命令 | 快捷键 | 功能 |
|------|--------|------|
| `run [args...]` | `r` | 运行程序（可指定参数） |
| `break <line\|function>` | `b` | 在指定行号或函数处设置断点 |
| `lsbreak` | `lb` | 列出所有断点 |
| `rmbreak <id>` | `rb` | 删除指定 ID 的断点 |
| `next` | `n` | 执行下一条语句（不进入函数） |
| `step` | `s` | 执行下一条语句（进入函数） |
| `continue` | `c` | 继续执行直到下一断点 |
| `print <expression>` | `p` | 计算并输出表达式的值 |
| `backtrace` | `bt` | 显示调用栈跟踪 |
| `optimizer [on\|off]` | `o` | 启用/禁用优化器 |
| `help` | `h` | 显示帮助信息 |
| `quit` | `q` | 退出调试器 |

**调试工作流示例**：

```bash
# 启动调试器（指定栈大小）
cs_dbg --stack-resize 10240 program.csc

# 设置断点
> break 10           # 在第 10 行设置断点
> break myFunction   # 在 myFunction 函数入口设置断点
> lsbreak            # 列出所有已设置的断点

# 开始调试
> run                # 开始运行程序
> n                  # 单步执行（不进入函数）
> s                  # 单步执行（进入函数）
> p x                # 打印变量 x 的值
> p x + y            # 计算并打印 x + y

# 查看调用栈
> bt                 # 显示当前调用栈

# 继续执行
> c                  # 继续运行到下一断点

# 清理与退出
> rmbreak 1          # 删除 ID 为 1 的断点
> q                  # 退出调试器
```

### Q: 如何在代码中添加断点？

**A**: CovScript 没有内置断点语句，但可以使用：

```covscript
# 手动暂停（需要用户输入）
function debugBreakpoint(message)
    system.out.println("断点: " + message)
    system.out.print("按 Enter 继续...")
    system.in.getline()
end

# 使用
debugBreakpoint("检查 x 的值")
system.out.println("x = " + to_string(x))
```

### Q: 如何进行单元测试？

**A**: 参考 [3.11 库开发最佳实践](./ecosystem/11-best-practices.md) 的测试框架示例。

---

## 疑难杂症

### Q: 中文输出乱码怎么办？

**A**: 在文件开头添加字符集声明：

```covscript
@charset: utf8
```

Windows 上可以使用 GBK：

```covscript
@charset: gbk
```

### Q: 为什么我的协程没有执行？

**A**: 协程需要手动调度。确保使用 `fiber.resume()` 恢复执行：

```covscript
var f = fiber.create([]() {
    system.out.println("Hello from fiber")
})

f.resume()  # 必须调用才会执行
```

### Q: 类成员函数中无法访问其他成员怎么办？

**A**: 使用 `this` 关键字：

```covscript
class MyClass
    var data = 10
    
    function process()
        system.out.println(this.data)  # 使用 this
    end
end
```

### Q: 为什么 `new` 关键字后需要加 `{}`？

**A**: `{}` 是参数列表的语法。仅在 ECS 中支持：

- **CSC**：`new MyClass` 创建对象（不支持 `{}` 语法）
- **ECS**：`new MyClass{arg1, arg2}` 创建对象并传递参数

```covscript
# CSC
var obj = new MyClass

# ECS
var obj = new MyClass{"Alice", 25}
```

### Q: 如何处理 "Stack overflow" 错误？

**A**: 
1. 检查是否有无限递归
2. 减少递归深度
3. 使用迭代代替递归

```covscript
# 递归（可能栈溢出）
function factorial(n)
    if n <= 1
        return 1
    end
    return n * factorial(n - 1)
end

# 迭代（更安全）
function factorial_iter(n)
    var result = 1
    for i = 2, i <= n, ++i
        result *= i
    end
    return result
end
```

### Q: ECS 编译失败怎么办？

**A**: 
1. 检查语法：`ecs -c program.ecs`
2. 清除缓存：删除 `.ecs_cache/` 目录
3. 确认 CSC 已正确安装
4. 查看详细错误信息

### Q: 如何报告 Bug？

**A**: 
1. 在 GitHub 提交 Issue：https://github.com/covscript/covscript/issues
2. 提供以下信息：
   - CovScript 版本（`cs -v`）
   - 操作系统和版本
   - 完整的错误信息
   - 最小可复现示例

---

## 更多帮助

- **官方文档**：https://manual.covscript.org.cn/
- **GitHub 仓库**：https://github.com/covscript/covscript
- **示例代码**：https://github.com/covscript/covscript-example
- **本地文档**：
  - [壹 · 引言](./intro/README.md)
  - [贰 · 语法基础](./syntax/README.md)
  - [叁 · 生态系统与扩展库](./ecosystem/README.md)

---

**最后更新日期**：2025-12-10

如果你有其他问题，欢迎在 GitHub 提交 Issue 或 Pull Request！
