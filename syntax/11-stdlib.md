# 2.11 标准库

CovScript 提供了丰富的标准库，涵盖输入输出、字符串操作、数学运算、系统操作等多个方面。

**重要说明：** 标准库 API 在 ECS 和 CSC 中基本一致，除非特别标注，否则所有示例在两个版本中都可以使用。标准库的具体实现可以参考 [type_ext.cpp](https://github.com/covscript/covscript/blob/master/sources/instance/type_ext.cpp)。

## 2.11.1 输入输出

### system.out - 标准输出

```covscript
# 输出文本
system.out.print("Hello, ")
system.out.println("World!")

# 输出数字
var num = 42
system.out.println(num)

# 输出多个值（通过拼接）
var name = "Alice"
var age = 25
system.out.println("Name: " + name + ", Age: " + to_string(age))

# 格式化输出
system.out.print("Loading")
for i = 0, i < 3, ++i
    system.out.print(".")
    runtime.delay(500)
end
system.out.println(" Done!")
```

### system.in - 标准输入

```covscript
# 读取一行输入
system.out.print("Enter your name: ")
var name = system.in.getline()
system.out.println("Hello, " + name + "!")

# 读取数字
system.out.print("Enter a number: ")
var input = system.in.getline()
var num = to_integer(input)
system.out.println("You entered: " + to_string(num))

# 读取多行输入
system.out.println("Enter text (empty line to finish):")
var lines = new list
loop
    var line = system.in.getline()
    if line.empty()
        break
    end
    lines.push_back(line)
end
```

### iostream - 文件输入输出

```covscript
# 打开文件进行读取
var file = iostream.fstream("data.txt", iostream.openmode.in)

# 读取文件内容
loop
    var line = file.getline()
    if file.eof()
        break
    end
    system.out.println(line)
end

file.close()

# 写入文件
var outFile = iostream.fstream("output.txt", iostream.openmode.out)
outFile.println("First line")
outFile.println("Second line")
outFile.println("Third line")
outFile.close()

# 追加模式
var appendFile = iostream.fstream("log.txt", iostream.openmode.app)
appendFile.println("New log entry")
appendFile.close()
```

## 2.11.2 字符串操作

CovScript 的字符串类型提供了丰富的内置方法，支持常见的字符串操作。

### 基本字符串方法

```covscript
var str = "Hello, World!"

# 获取长度（size 是属性，不是方法）
var len = str.size
system.out.println("Length: " + to_string(len))

# 获取字符（使用下标访问）
var firstChar = str[0]      # 'H'
var lastChar = str[str.size - 1]  # '!'

# 字符串切片（substr(起始位置, 长度)）
var sub = str.substr(0, 5)  # "Hello" - 从位置0开始，提取5个字符
var sub2 = str.substr(7, 5) # "World" - 从位置7开始，提取5个字符
system.out.println("Substring: " + sub)

# 查找子串（find(子串, 起始位置)，返回位置或-1）
var pos = str.find("World", 0)
if pos != -1
    system.out.println("Found at position: " + to_string(pos))
else
    system.out.println("Not found")
end

# 字符串比较（使用 == 运算符）
var str1 = "abc"
var str2 = "abc"
if str1 == str2
    system.out.println("Strings are equal")
end

# 检查字符串是否为空
var empty = ""
if empty.empty()
    system.out.println("String is empty")
end
```

### 字符串拼接和构建

```covscript
# 使用 + 运算符
var greeting = "Hello" + ", " + "World" + "!"

# 构建复杂字符串
var parts = {"one", "two", "three"}

# 使用 join 拼接数组
var result = parts.join(", ")
system.out.println(result)  # "one, two, three"
```

### 字符串转换

```covscript
# 大小写转换
var text = "hello"
var upper = text.to_upper()
system.out.println(upper)  # "HELLO"

var text2 = "WORLD"
var lower = text2.to_lower()
system.out.println(lower)  # "world"

# 字符串裁剪空白
var text3 = "  hello  "
var trimmed = text3.trim()
system.out.println("'" + trimmed + "'")  # "hello"

# 字符串分割
var csv = "apple,banana,orange"
var fruits = csv.split({','})
foreach fruit in fruits
    system.out.println(fruit)
end
```

## 2.11.3 类型转换

### 转换为字符串

```covscript
# 数字转字符串
var num = 42
var str = to_string(num)
system.out.println("String: " + str)

# 布尔转字符串
var flag = true
var boolStr = to_string(flag)
system.out.println("Boolean: " + boolStr)

# 其他类型转字符串
var arr = {1, 2, 3}
var arrStr = to_string(arr)
```

### 转换为数字

```covscript
# 字符串转整数
var str = "123"
var num = to_integer(str)
system.out.println("Integer: " + to_string(num))

# 字符串转浮点数
var floatStr = "3.14"
var floatNum = to_number(floatStr)
system.out.println("Float: " + to_string(floatNum))

# 处理转换错误
try
    var invalid = to_integer("not a number")
catch e
    system.out.println("Conversion error: " + e)
end
```

### 类型检查

```covscript
var x = 42
var y = "hello"
var z = {1, 2, 3}

# 使用 typeid
if typeid x == typeid 0
    system.out.println("x is a number")
end

if typeid y == typeid ""
    system.out.println("y is a string")
end

# 使用 type 函数
system.out.println("Type of x: " + to_string(type(x)))
system.out.println("Type of y: " + to_string(type(y)))
```

## 2.11.4 数学运算

### 基本数学函数

```covscript
# 绝对值
var abs1 = math.abs(-10)
system.out.println("abs(-10) = " + to_string(abs1))

# 平方根
var sqrt1 = math.sqrt(16)
system.out.println("sqrt(16) = " + to_string(sqrt1))

# 幂运算
var pow1 = math.pow(2, 10)
system.out.println("pow(2, 10) = " + to_string(pow1))

# 三角函数
var sin1 = math.sin(0)
var cos1 = math.cos(0)
var tan1 = math.tan(0)

# 对数
var ln1 = math.ln(10)         # 自然对数
var log10_1 = math.log10(100)  # 以10为底的对数
var log1 = math.log(2, 8)     # 以2为底8的对数
```

### 数学常量

```covscript
# 数学常量
var pi = math.constants.pi
var e = math.constants.e

system.out.println("PI = " + to_string(pi))
system.out.println("E = " + to_string(e))

# 使用常量
var circumference = 2 * pi * 5
var area = pi * 5 * 5

system.out.println("Circumference: " + to_string(circumference))
system.out.println("Area: " + to_string(area))
```

### 随机数

```covscript
# 生成随机浮点数
var random1 = math.rand(0, 1)  # 0到1之间的随机浮点数
system.out.println("Random float: " + to_string(random1))

# 生成指定范围的随机整数
var dice = math.randint(1, 6)
system.out.println("Dice roll: " + to_string(dice))

# 生成随机浮点数
var random2 = math.rand(10.5, 20.5)
system.out.println("Random in range: " + to_string(random2))
```

## 2.11.5 系统操作

### system.path - 路径操作

```covscript
# 检查文件是否存在
var exists = system.path.exist("myfile.txt")
if exists
    system.out.println("File exists")
end

# 检查是否为目录
var isDir = system.path.is_dir("myfolder")
if isDir
    system.out.println("It's a directory")
end

# 扫描目录
var files = system.path.scan(".")
foreach file in files
    system.out.println("File: " + file)
end

# 获取文件信息
var info = system.path.info("myfile.txt")
# 根据实际API使用文件信息
```

### system.file - 文件操作

```covscript
# 删除文件
system.file.remove("temp.txt")

# 重命名文件
system.file.rename("old.txt", "new.txt")

# 复制文件（如果支持）
# system.file.copy("source.txt", "dest.txt")

# 创建目录
system.path.mkdir("new_folder")

# 删除目录
system.path.rmdir("old_folder")
```

### 环境变量

```covscript
# 获取环境变量
var home = system.env.get("HOME")
system.out.println("Home directory: " + home)

# 设置环境变量（如果支持）
# system.env.set("MY_VAR", "value")
```

## 2.11.6 运行时操作（runtime）

### 时间操作

```covscript
# 获取当前时间戳
var timestamp = runtime.time()
system.out.println("Timestamp: " + to_string(timestamp))

# 暂停执行
system.out.println("Waiting 2 seconds...")
runtime.delay(2000)  # 毫秒
system.out.println("Done!")

# 计时器
var startTime = runtime.time()
# 执行某些操作
for i = 0, i < 1000000, ++i
    var x = i * i
end
var endTime = runtime.time()
var elapsed = endTime - startTime
system.out.println("Elapsed: " + to_string(elapsed) + "ms")
```

### 程序控制

```covscript
# 退出程序
function exitProgram(code)
    system.out.println("Exiting with code: " + to_string(code))
    runtime.exit(code)
end

# 获取命令行参数
# 注意：context 参数需要显式传递给需要访问上下文的函数
function showArgs(context)
    var args = context.cmd_args
    foreach arg in args
        system.out.println("Arg: " + arg)
    end
end

# 执行系统命令
var result = system.run("ls -l")
system.out.println("Exit code: " + to_string(result))
```

## 2.11.7 上下文操作（context）

### 导入路径管理

```covscript
# 获取当前导入路径
var path = runtime.get_import_path()
system.out.println("Import path: " + path)

# 动态导入模块
# 注意：context 参数需要显式传递给需要访问上下文的函数
function loadModule(context, path)
    var module = context.source_import(path)
    return module
end
```

### 变量管理

```covscript
# 获取全局变量（根据实际API）
# var globals = context.vars()

# 动态访问变量
# var value = context.get("variableName")

# 动态设置变量
# context.set("variableName", 42)
```

## 2.11.8 协程（fiber）

协程提供轻量级的并发能力。

```covscript
import fiber

# 创建协程
var f = fiber.create([]() {
    for i=0,i < 5,++i
        system.out.println("Fiber: " + to_string(i))
        fiber.yield()  # 让出执行权
    end
})

# 运行协程
for i=0,i < 5,++i
    system.out.println("Main: " + to_string(i))
    fiber.resume(f)  # 恢复协程
end

# 带参数的协程
var task = fiber.create([](name) {
    for i=0,i < 3,++i
        system.out.println(name + ": " + to_string(i))
        fiber.yield()
    end
})

fiber.resume(task, "Task1")
```

**提示**：关于协程和异步编程的详细内容，请参阅 [2.13 异步编程与协程](./13-asyncio.md) 章节。

## 2.11.9 编译时求值

### @begin/@end 块

编译时执行的代码块。

```covscript
# 编译时计算常量
@begin
var compiledValue = 10 * 10
@end

# 在运行时使用编译时计算的值
constant BUFFER_SIZE = compiledValue

# 编译时生成代码
@begin
for i=0,i < 5,++i
    # 生成变量声明
    # var var${i} = ${i * 10}
end
@end
```

## 2.11.10 特殊指令

### @charset - 字符集设置

```covscript
# 设置源文件字符集
@charset "utf-8"

# 现在可以使用UTF-8字符
var greeting = "你好，世界！"
system.out.println(greeting)

var emoji = "😀🎉"
system.out.println(emoji)
```

## 2.11.11 实用工具函数

### 集合工具

```covscript
# 数组/列表工具
function contains(container, item)
    foreach element in container
        if element == item
            return true
        end
    end
    return false
end

function indexOf(container, item)
    var index = 0
    foreach element in container
        if element == item
            return index
        end
        index += 1
    end
    return -1
end

function reverse(container)
    var result = new list
    for i = container.size - 1, i >= 0, --i
        result.push_back(container[i])
    end
    return result
end
```

### 字符串工具

```covscript
# 字符串分割（使用内置函数）
var text = "apple,banana,orange"
var items = text.split({','})
foreach item in items
    system.out.println(item)
end

# 数组连接（使用内置函数）
var words = {"Hello", "World", "!"}
var sentence = words.join(" ")
system.out.println(sentence)

# 字符串裁剪（使用内置函数）
var text2 = "  hello world  "
var trimmed = text2.trim()
system.out.println("'" + trimmed + "'")

# 字符串查找和替换
var str = "Hello World"
var pos = str.find("World", 0)
if pos != -1
    system.out.println("Found at: " + to_string(pos))
end

# 字符串子串
var sub = str.substr(0, 5)
system.out.println("Substring: " + sub)
```

## 2.11.12 更多实用示例

### CSV 文件读写

```covscript
# 读取 CSV 文件
function readCSV(filename)
    var records = new list
    var file = iostream.fstream(filename, iostream.openmode.in)
    
    try
        var header = file.getline()
        var headers = split(header, ',')
        
        loop
            var line = file.getline()
            if file.eof()
                break
            end
            
            var values = split(line, ',')
            var record = new hash_map
            
            for i = 0, i < headers.size && i < values.size, ++i
                record.insert(trim(headers[i]), trim(values[i]))
            end
            
            records.push_back(record)
        end
    catch e
        system.out.println("读取 CSV 错误: " + e)
    finally
        file.close()
    end
    
    return records
end

# 写入 CSV 文件
function writeCSV(filename, data, headers)
    var file = iostream.fstream(filename, iostream.openmode.out)
    
    try
        # 写入头部
        file.println(headers.join(","))
        
        # 写入数据
        foreach record in data
            var values = new array
            foreach header in headers
                if record.exist(header)
                    values.push_back(to_string(record[header]))
                else
                    values.push_back("")
                end
            end
            file.println(values.join(","))
        end
    catch e
        system.out.println("写入 CSV 错误: " + e)
    finally
        file.close()
    end
end
```

### 进度条显示

```covscript
# 显示文本进度条
function showProgress(current, total, width)
    var percent = (current * 100) / total
    var filled = (current * width) / total
    
    var bar = "["
    for i = 0, i < width, ++i
        if i < filled
            bar += "="
        else
            bar += " "
        end
    end
    bar += "] " + to_string(percent) + "%"
    
    system.out.print("\r" + bar)
    
    if current >= total
        system.out.println("")
    end
end

# 使用示例
var total = 100
for i = 0, i <= total, ++i
    showProgress(i, total, 50)
    runtime.delay(50)
end
```

### 配置文件管理

```covscript
# INI 格式配置文件读取
function readINI(filename)
    var config = new hash_map
    var currentSection = "default"
    var file = iostream.fstream(filename, iostream.openmode.in)
    
    try
        loop
            var line = file.getline()
            if file.eof()
                break
            end
            
            line = line.trim()
            
            # 跳过空行和注释
            if line.empty() || line[0] == '#' || line[0] == ';'
                continue
            end
            
            # 检查是否是节标题
            if line[0] == '[' && line[line.size - 1] == ']'
                currentSection = line.substr(1, line.size - 2)
                if !config.exist(currentSection)
                    config.insert(currentSection, new hash_map)
                end
            else
                # 解析键值对
                var eqPos = line.find("=", 0)
                if eqPos != -1
                    var key = line.substr(0, eqPos).trim()
                    var value = line.substr(eqPos + 1, line.size - eqPos - 1).trim()
                    
                    if !config.exist(currentSection)
                        config.insert(currentSection, new hash_map)
                    end
                    
                    config[currentSection].insert(key, value)
                end
            end
        end
    catch e
        system.out.println("读取 INI 错误: " + e)
    finally
        file.close()
    end
    
    return config
end
```

### 简单的日志系统

```covscript
class Logger
    var logFile = null
    var logLevel = "INFO"
    var levels = {"DEBUG": 0, "INFO": 1, "WARNING": 2, "ERROR": 3}
    
    function construct(filename)
        this.logFile = iostream.fstream(filename, iostream.openmode.app)
    end
    
    function setLevel(level)
        this.logLevel = level
    end
    
    function log(level, message)
        if this.levels[level] >= this.levels[this.logLevel]
            var timestamp = runtime.time()
            var logEntry = "[" + to_string(timestamp) + "] [" + level + "] " + message
            
            # 写入文件
            this.logFile.println(logEntry)
            this.logFile.flush()
            
            # 同时输出到控制台
            system.out.println(logEntry)
        end
    end
    
    function debug(message)
        this.log("DEBUG", message)
    end
    
    function info(message)
        this.log("INFO", message)
    end
    
    function warning(message)
        this.log("WARNING", message)
    end
    
    function error(message)
        this.log("ERROR", message)
    end
    
    function close()
        if this.logFile != null
            this.logFile.close()
        end
    end
end

# 使用日志系统
var logger = new Logger{"app.log"}
logger.setLevel("INFO")

logger.debug("这条不会显示")
logger.info("应用程序启动")
logger.warning("这是一个警告")
logger.error("发生错误")

logger.close()
```

## 标准库使用最佳实践

1. **导入需要的模块**：只导入实际使用的模块
2. **错误处理**：文件和系统操作要处理异常
3. **资源管理**：及时关闭文件和其他资源
4. **性能考虑**：避免频繁的类型转换和字符串拼接
5. **跨平台兼容**：注意路径分隔符和系统差异
6. **使用工具函数**：为常用操作创建可复用的工具函数
7. **日志记录**：在关键位置添加日志，便于调试和监控

```covscript
# 好的实践示例
function readConfig(filename)
    var file = null
    var config = new hash_map
    
    try
        # 检查文件是否存在
        if !system.path.exist(filename)
            throw "Config file not found"
        end
        
        # 打开文件
        file = iostream.fstream(filename, iostream.openmode.in)
        
        # 读取配置
        loop
            var line = file.getline()
            if file.eof()
                break
            end
            
            # 跳过空行和注释
            var trimmed = line.trim()
            if trimmed.empty() || trimmed[0] == '#'
                continue
            end
            
            # 解析配置行
            var parts = trimmed.split({'='})
            if parts.size == 2
                config.insert(parts[0].trim(), parts[1].trim())
            end
        end
        
        # 关闭文件
        file.close()
        return config
        
    catch e
        # 确保文件被关闭
        if file != null
            file.close()
        end
        system.out.println("Error reading config: " + e)
        return null
    end
end
```
