# 3.11 库开发最佳实践

本章介绍开发高质量 CovScript 库的最佳实践，涵盖模块设计、代码组织、错误处理、性能优化等方面。

## 3.11.1 模块化设计

### 使用 Package 组织代码

使用 `package` 关键字创建可复用的代码库：

```covscript
# mylib.csp
package mylib
    # 版本信息
    constant VERSION = "1.0.0"
    constant AUTHOR = "Your Name"
    
    # 私有函数（使用 _ 前缀命名）
    function _internalHelper(x)
        return x * 2
    end
    
    # 公共 API
    function publicFunction(x)
        if x == null
            throw "参数不能为 null"
        end
        return _internalHelper(x) + 1
    end
    
    # 工具类
    class MyTool
        var data = null
        
        function construct(initialData)
            this.data = initialData
        end
        
        function process()
            return _internalHelper(this.data)
        end
    end
    
    # 常量配置
    namespace config
        constant MAX_SIZE = 1000
        constant TIMEOUT = 5000
        constant DEBUG = false
    end
end
```

### 命名空间分层

使用嵌套命名空间组织复杂功能：

```covscript
package graphics
    # 基础图形
    namespace shapes
        class Circle
            var radius = 0
            function construct(r)
                this.radius = r
            end
            function area()
                return math.constants.pi * this.radius * this.radius
            end
        end
        
        class Rectangle
            var width = 0
            var height = 0
            function construct(w, h)
                this.width = w
                this.height = h
            end
            function area()
                return this.width * this.height
            end
        end
    end
    
    # 颜色处理
    namespace colors
        constant RED = {255, 0, 0}
        constant GREEN = {0, 255, 0}
        constant BLUE = {0, 0, 255}
        
        function rgb(r, g, b)
            return {r, g, b}
        end
    end
end
```

## 3.11.2 文档化

### 函数文档注释

为公共 API 添加详细的文档注释：

```covscript
# @brief 计算两个数的和
# @param a 第一个数字（number）
# @param b 第二个数字（number）
# @return 两数之和（number）
# @example
#   var result = add(10, 20)
#   system.out.println(result)  # 输出: 30
function add(a, b)
    return a + b
end

# @brief 从文件读取配置
# @param filename 配置文件路径（string）
# @return 配置对象（hash_map），如果失败返回 null
# @throws 文件不存在或格式错误时抛出异常
# @example
#   var config = readConfig("app.conf")
#   if config != null
#       var port = config["port"]
#   end
function readConfig(filename)
    if !system.path.exist(filename)
        throw "配置文件不存在: " + filename
    end
    
    # 读取和解析逻辑
    var config = new hash_map
    # ...
    return config
end
```

### 包级文档（README）

为每个包创建 README.md：

```markdown
# MyLib - CovScript 工具库

## 简介

MyLib 是一个提供常用工具函数的 CovScript 库。

## 安装

```bash
cspkg install mylib --yes
```

## 快速开始

```covscript
import mylib

var result = mylib.publicFunction(42)
system.out.println(result)
```

## API 文档

### mylib.publicFunction(x)

计算 x 的两倍加一。

**参数**：
- `x` (number) - 输入数字

**返回值**：
- (number) - 计算结果

**示例**：
```covscript
var result = mylib.publicFunction(10)
# result 为 21
```

## 许可证

MIT License
```

## 3.11.3 错误处理

### 输入验证

在函数入口验证所有参数：

```covscript
function divide(a, b)
    # 参数类型检查
    if typeid a != typeid 0 || typeid b != typeid 0
        throw "divide: 参数必须是数字类型"
    end
    
    # 参数值检查
    if b == 0
        throw "divide: 除数不能为零"
    end
    
    return a / b
end

# 使用示例
try
    var result = divide(10, 2)
    system.out.println(result)
catch e
    system.out.println("错误: " + e)
end
```

### 优雅的错误恢复

提供合理的默认值和错误恢复机制：

```covscript
function safeReadFile(filename, defaultContent)
    try
        if !system.path.exist(filename)
            system.out.println("警告: 文件不存在，使用默认内容")
            return defaultContent
        end
        
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
        
    catch e
        system.out.println("读取文件失败: " + e)
        return defaultContent
    end
end

# 使用示例
var content = safeReadFile("config.txt", "# 默认配置")
```

### 自定义异常类

创建特定的异常类型：

```covscript
class ValidationError
    var message = ""
    var field = ""
    
    function construct(msg, fld)
        this.message = msg
        this.field = fld
    end
    
    function toString()
        return "ValidationError: " + this.field + " - " + this.message
    end
end

function validateUser(user)
    if !user.exist("name") || user["name"].empty()
        throw new ValidationError{"Name is required", "name"}
    end
    
    if !user.exist("age") || user["age"] < 0
        throw new ValidationError{"Age must be positive", "age"}
    end
    
    return true
end

# 使用示例
var user = new hash_map
user.insert("name", "Alice")
user.insert("age", 25)

try
    validateUser(user)
    system.out.println("验证通过")
catch e
    if typeid e == typeid new ValidationError{"", ""}
        system.out.println(e.toString())
    else
        system.out.println("未知错误: " + to_string(e))
    end
end
```

## 3.11.4 版本管理与兼容性

### 版本号约定

使用语义化版本（Semantic Versioning）：

```covscript
package mylib
    # 版本信息
    constant VERSION_MAJOR = 1
    constant VERSION_MINOR = 2
    constant VERSION_PATCH = 3
    constant VERSION = to_string(VERSION_MAJOR) + "." + 
                      to_string(VERSION_MINOR) + "." + 
                      to_string(VERSION_PATCH)
    
    function getVersion()
        return VERSION
    end
    
    # 检查兼容性
    function checkCompatibility(requiredVersion)
        var parts = requiredVersion.split({'.'})
        var reqMajor = to_integer(parts[0])
        
        # 主版本号必须匹配
        if reqMajor != VERSION_MAJOR
            throw "版本不兼容: 需要 " + requiredVersion + "，当前 " + VERSION
        end
        
        return true
    end
end
```

### 特性检测

提供特性检测机制而非硬编码版本检查：

```covscript
namespace features
    # 检测是否支持某个特性
    function hasAsyncSupport()
        try
            # 尝试创建协程
            var f = fiber.create([]() {})
            return true
        catch e
            return false
        end
    end
    
    function hasFileWatchSupport()
        try
            # 检查文件监控 API
            return typeid system.file.watch != typeid null
        catch e
            return false
        end
    end
    
    # 根据特性选择实现
    function selectImplementation()
        if hasAsyncSupport()
            return "async_impl"
        else
            return "sync_impl"
        end
    end
end
```

## 3.11.5 性能优化

### 避免不必要的对象创建

```covscript
# 不好的做法：频繁创建对象
function badExample(n)
    for i = 0, i < n, ++i
        var temp = new list  # 每次循环都创建
        temp.push_back(i)
    end
end

# 好的做法：复用对象
function goodExample(n)
    var temp = new list  # 只创建一次
    for i = 0, i < n, ++i
        temp.clear()
        temp.push_back(i)
    end
end
```

### 缓存计算结果

```covscript
class ExpensiveCalculator
    var cache = new hash_map
    
    function fibonacci(n)
        # 检查缓存
        if cache.exist(to_string(n))
            return cache[to_string(n)]
        end
        
        # 计算结果
        var result = 0
        if n <= 1
            result = n
        else
            result = fibonacci(n - 1) + fibonacci(n - 2)
        end
        
        # 存入缓存
        cache.insert(to_string(n), result)
        return result
    end
end
```

### 使用适当的数据结构

```covscript
# 频繁查找：使用 hash_map
var userMap = new hash_map
userMap.insert("alice", {name: "Alice", age: 25})
userMap.insert("bob", {name: "Bob", age: 30})

var alice = userMap["alice"]  # O(1) 查找

# 顺序遍历：使用 list
var todoList = new list
todoList.push_back("任务1")
todoList.push_back("任务2")

foreach task in todoList
    system.out.println(task)
end

# 固定大小：使用 array
var fixedData = {1, 2, 3, 4, 5}
var firstItem = fixedData[0]
```

## 3.11.6 测试

### 单元测试框架

创建简单的测试框架：

```covscript
# test_framework.csc
namespace test
    var passed = 0
    var failed = 0
    var tests = new list
    
    function assert(condition, message)
        if condition
            ++passed
            system.out.println("✓ " + message)
        else
            ++failed
            system.out.println("✗ " + message)
        end
    end
    
    function assertEqual(expected, actual, message)
        if expected == actual
            ++passed
            system.out.println("✓ " + message)
        else
            ++failed
            system.out.println("✗ " + message + 
                             " (期望: " + to_string(expected) + 
                             ", 实际: " + to_string(actual) + ")")
        end
    end
    
    function runTest(name, testFunc)
        system.out.println("\n运行测试: " + name)
        try
            testFunc()
        catch e
            ++failed
            system.out.println("✗ 测试异常: " + e)
        end
    end
    
    function summary()
        var total = passed + failed
        system.out.println("\n" + ("="):repeat(50))
        system.out.println("测试结果: " + to_string(passed) + "/" + 
                          to_string(total) + " 通过")
        if failed == 0
            system.out.println("所有测试通过！")
        else
            system.out.println(to_string(failed) + " 个测试失败")
        end
        system.out.println(("="):repeat(50))
    end
end
```

### 测试示例

```covscript
# test_mylib.csc
import test
import mylib

# 测试基本功能
test.runTest("publicFunction 基本测试", []() {
    test.assertEqual(21, mylib.publicFunction(10), "10 的结果应该是 21")
    test.assertEqual(1, mylib.publicFunction(0), "0 的结果应该是 1")
})

# 测试边界条件
test.runTest("publicFunction 边界测试", []() {
    test.assertEqual(-999, mylib.publicFunction(-1000), "负数测试")
})

# 测试异常处理
test.runTest("publicFunction 异常测试", []() {
    try
        mylib.publicFunction(null)
        test.assert(false, "应该抛出异常")
    catch e
        test.assert(true, "正确抛出异常")
    end
})

# 显示测试摘要
test.summary()
```

## 3.11.7 代码风格

### 命名约定

```covscript
# 常量：大写下划线
constant MAX_CONNECTIONS = 100
constant API_VERSION = "v1"

# 类名：大驼峰
class UserManager
    # 成员变量：小驼峰
    var userName = ""
    var userId = 0
    
    # 方法名：小驼峰
    function getUserInfo()
        return this.userName
    end
end

# 函数名：小驼峰
function calculateTotal(items)
    return 0
end

# 私有函数/变量：下划线前缀
function _internalHelper()
    return 0
end
```

### 代码组织

```covscript
# 1. 预处理指令
@charset: utf8

# 2. 导入语句
import network
import csdbc

# 3. 常量定义
constant VERSION = "1.0.0"
constant DEBUG = false

# 4. 工具函数
function _helperFunction()
    return 0
end

# 5. 类定义
class MyClass
    # ...
end

# 6. 公共 API
function publicAPI()
    # ...
end

# 7. 主程序（如果是可执行脚本）
function main()
    # ...
end

# 8. 程序入口
if context.cmd_args.size > 0
    main()
end
```

## 3.11.8 发布清单

发布库之前的检查清单：

- [ ] **文档完整**
  - [ ] README.md 包含安装、使用说明
  - [ ] API 文档齐全
  - [ ] 包含示例代码

- [ ] **代码质量**
  - [ ] 所有公共 API 有文档注释
  - [ ] 代码风格一致
  - [ ] 无明显性能问题

- [ ] **测试覆盖**
  - [ ] 核心功能有单元测试
  - [ ] 边界情况已测试
  - [ ] 异常处理已测试

- [ ] **版本信息**
  - [ ] 版本号符合语义化版本
  - [ ] CHANGELOG.md 更新
  - [ ] Git 标签已创建

- [ ] **许可证**
  - [ ] LICENSE 文件存在
  - [ ] 版权信息明确

- [ ] **依赖管理**
  - [ ] package.json（如有）正确配置
  - [ ] 依赖版本明确

## 参考资料

### 官方资源
- **库开发示例**：https://github.com/covscript
- **CNI 接口文档**：https://github.com/covscript/covscript/blob/master/docs/zh-cn/190201_cni.md
- **扩展开发指南**：https://github.com/covscript/covscript/blob/master/docs/zh-cn/190101_extensions.md

### 优秀的开源库
- **network**：https://github.com/covscript/covscript-network
- **csdbc**：https://github.com/covscript/csdbc
- **imgui**：https://github.com/covscript/covscript-imgui
- **covanalysis**：https://github.com/covscript/covanalysis

学习这些库的代码组织和 API 设计，可以帮助你编写更好的 CovScript 库。