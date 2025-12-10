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
sudo apt-get install libc6 libstdc++6
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

### Q: 为什么字符串拼接时数字需要转换？

**A**: CovScript 不会自动将数字转为字符串，需要使用 `to_string()`:

```covscript
var age = 25
system.out.println("Age: " + to_string(age))  # 正确
# system.out.println("Age: " + age)           # 错误
```

### Q: 如何定义多行字符串？

**A**: 使用转义字符 `\n`：

```covscript
var multiline = "第一行\n第二行\n第三行"
system.out.println(multiline)
```

CovScript 不支持三引号多行字符串。

### Q: 数组和列表有什么区别？

**A**: 
- **数组 (Array)**：`{1, 2, 3}`，固定大小（虽然可以通过索引自动扩容）
- **列表 (List)**：`new list`，动态大小，提供 `push_back`、`pop_back` 等方法

推荐使用列表进行动态操作。

### Q: 如何实现类似 Python 的列表推导式？

**A**: CovScript 没有内置列表推导式，但可以用循环实现：

```covscript
# 生成 1 到 10 的平方
var squares = new list
for i = 1, i <= 10, ++i
    squares.push_back(i * i)
end
```

### Q: 函数参数是值传递还是引用传递？

**A**: 
- 基本类型（数字、布尔、字符）：**值传递**
- 对象类型（数组、列表、类实例）：**引用传递**

```covscript
function modifyList(lst)
    lst.push_back(100)  # 会修改原列表
end

function modifyNumber(num)
    num = 100  # 不会修改原变量
end
```

在 ECS 中，可以使用引用参数 `=` 实现值类型的引用传递。

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

**A**: 
- **CovScript 3**：在类体内定义 `construct` 函数，但 `new` 不能传参
- **ECS**：`new` 和 `gcnew` 可以传参

```covscript
class Person
    var name = ""
    var age = 0
    
    function construct(n, a)
        this.name = n
        this.age = a
    end
end

# CovScript 3
var p = new Person{}
# 需要手动调用 construct 或使用其他方式初始化

# ECS
var p = new Person{"Alice", 25}
```

---

## 标准库

### Q: 如何读取整个文件的内容？

**A**: 使用循环读取：

```covscript
function readFile(filename)
    var file = iostream.fstream(filename, iostream.openmode.in)
    var content = ""
    
    loop
        var line = file.getline()
        if file.eof()
            break
        end
        content += line + "\n"
    end
    
    file.close()
    return content
end
```

### Q: 如何获取命令行参数？

**A**: 使用 `context.cmd_args`（需要在可以访问上下文的地方）：

```covscript
# 在主程序中
if context.cmd_args.size > 1
    var arg1 = context.cmd_args[1]
    system.out.println("第一个参数: " + arg1)
end
```

或使用 `argparse` 扩展库进行更复杂的参数解析。

### Q: 如何生成随机数？

**A**: 

```covscript
# 随机整数（1-100）
var num = math.randint(1, 100)

# 随机浮点数（0.0-1.0）
var float = math.rand(0, 1)

# 随机浮点数（指定范围）
var float2 = math.rand(10.5, 20.5)
```

### Q: 如何遍历目录中的文件？

**A**: 使用 `system.path.scan()`：

```covscript
var files = system.path.scan(".")
foreach file in files
    if !system.path.is_dir(file)
        system.out.println("文件: " + file)
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

**A**: 使用 `imgui_font.source_han_sans` 字体：

```covscript
import imgui

window_application([]() {
    add_font_extend_cn(imgui_font.source_han_sans)
    push_font(0)
    
    # GUI 代码
    text("你好，世界！")
    
    pop_font()
})
```

---

## 性能与优化

### Q: CovScript 的性能如何？

**A**: CovScript 是解释型语言，性能介于 Python 和 Lua 之间。对于 CPU 密集型任务，建议：
1. 使用 C++ 编写扩展模块
2. 减少不必要的对象创建
3. 使用合适的数据结构

### Q: 如何提高代码执行速度？

**A**: 
1. **避免频繁的类型转换**
2. **减少字符串拼接**（使用列表收集后一次性拼接）
3. **缓存计算结果**
4. **使用局部变量**而非全局变量
5. **选择合适的数据结构**（hash_map vs list）

### Q: 内存占用太高怎么办？

**A**: 
1. 及时关闭文件和资源
2. 清空不再使用的大型列表/数组
3. 避免循环引用
4. 使用 `gcnew` 代替 `new`（由垃圾回收器管理）

---

## 开发工具

### Q: 有没有 CovScript 的 IDE？

**A**: 推荐使用 **VSCode** + CovScript 扩展：
1. 安装 VSCode
2. 搜索并安装 "CovScript" 扩展（by mikecovlee）
3. 享受语法高亮和代码补全

### Q: 如何调试 CovScript 程序？

**A**: 使用 `cs_dbg` 调试器：

```bash
cs_dbg program.csc

# 调试器命令
break 10      # 在第 10 行设置断点
run           # 运行程序
step          # 单步执行
print x       # 打印变量 x 的值
continue      # 继续执行
quit          # 退出调试器
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

fiber.resume(f)  # 必须调用才会执行
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

**A**: 这是 CovScript 3 的语法要求。ECS 中可以使用 `new ClassName{args}`。

```covscript
# CovScript 3
var obj = new MyClass{}

# ECS
var obj = new MyClass{arg1, arg2}
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
3. 确认 CovScript 3 已正确安装
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

**最后更新日期**：2025-12-08

如果你有其他问题，欢迎在 GitHub 提交 Issue 或 Pull Request！
