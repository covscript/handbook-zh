# 2.7 模块与包

CovScript 提供了强大的模块系统，用于组织和复用代码。

## 2.7.1 包定义（package）

使用 `package` 关键字定义包，将相关的函数、类和变量组织在一起。

```covscript
# math_utils.csp
package math_utils
    # 常量
    constant PI = 3.14159265359
    constant E = 2.71828182846
    
    # 函数
    function square(x)
        return x * x
    end
    
    function cube(x)
        return x * x * x
    end
    
    function power(base, exponent)
        var result = 1
        for i=0, i < exponent, ++i
            result *= base
        end
        return result
    end
    
    function abs(x)
        if x < 0
            return -x
        end
        return x
    end
end
```

### 使用包

```covscript
# main.csc
import math_utils

# 使用包中的成员
var result1 = math_utils.square(5)
system.out.println(result1)  # 25

var result2 = math_utils.cube(3)
system.out.println(result2)  # 27

system.out.println(math_utils.PI)  # 3.14159265359
```

## 2.7.2 命名空间（namespace）

命名空间用于组织代码，避免命名冲突。

```covscript
# 定义命名空间
namespace geometry
    function calculateCircleArea(radius)
        return 3.14159 * radius * radius
    end
    
    function calculateRectangleArea(width, height)
        return width * height
    end
    
    namespace shape3d
        function calculateSphereVolume(radius)
            return (4.0 / 3.0) * 3.14159 * radius * radius * radius
        end
        
        function calculateCubeVolume(side)
            return side * side * side
        end
    end
end

# 使用命名空间
var area = geometry.calculateCircleArea(5)
system.out.println("Circle area: " + to_string(area))

var volume = geometry.shape3d.calculateSphereVolume(3)
system.out.println("Sphere volume: " + to_string(volume))
```

### 嵌套命名空间

```covscript
namespace company
    namespace department
        namespace engineering
            function getTotalEmployees()
                return 50
            end
            
            function getAverageSalary()
                return 80000
            end
        end
        
        namespace sales
            function getTotalEmployees()
                return 30
            end
            
            function getAverageSalary()
                return 60000
            end
        end
    end
end

# 访问嵌套命名空间
var engCount = company.department.engineering.getTotalEmployees()
system.out.println("Engineering employees: " + to_string(engCount))
```

## 2.7.3 导入模块（import）

### 导入整个模块

```covscript
# 导入正则表达式模块
import regex

# 使用正则表达式
var pattern = regex.build("\\d+")
var text = "The answer is 42"
var match = regex.match(text, pattern)

if match
    system.out.println("Found number!")
end
```

### 导入模块中的一个子空间

```covscript
# 仅导入 JSON 模块
import codec.json

# 使用模块
var data = new hash_map
data.insert("name", "Alice")
data.insert("age", 25)

var json_obj = json.from_var(data)
system.out.println(json.to_string(json_obj))
```

## 2.7.4 import as（别名导入）

使用 `as` 关键字为导入的模块指定别名。

```covscript
# 使用别名导入
import codec.json as j

var data = new hash_map
data.insert("key", "value")

# 使用别名
var json_obj = j.from_var(data)
system.out.println(json.to_string(json_obj))

# 导入多个模块并使用别名
import codec.base64.rfc4648 as b64
import codec.json as json_codec

var text = "Hello, World!"
var encoded = b64.encode(text)
system.out.println("Encoded: " + encoded)
```

## 2.7.5 using 语句

`using` 语句将命名空间或包的成员导入到当前作用域。

```covscript
# 不使用 using
var result1 = math.abs(-10)
var result2 = math.sqrt(16)

# 使用 using
using math

var result3 = abs(-10)    # 直接使用，无需前缀
var result4 = sqrt(16)

# using 命名空间
namespace utils
    function helper()
        return "Helper function"
    end
end

using utils
var msg = helper()
system.out.println(msg)
```

## 2.7.6 包的组织结构

### 创建可复用的包

```covscript
# string_utils.csp
package string_utils

# 字符串工具函数
function reverse(str)
    var result = ""
    for i = str.size - 1, i >= 0, --i
        result += str[i]
    end
    return result
end

function toUpperCase(str)
    var result = ""
    foreach ch in str
        if ch.isalpha()
            result += ch.toupper()
        else
            result += ch
        end
    end
    return result
end

function toLowerCase(str)
    var result = ""
    foreach ch in str
        if ch.isalpha()
            result += ch.tolower()
        else
            result += ch
        end
    end
    return result
end

function contains(str, substr)
    return str.find(substr) != -1
end

function split(str, delimiter)
    var result = new list
    var current = ""
    
    foreach ch in str
        if ch == delimiter
            if current.size > 0
                result.push_back(current)
                current = ""
            end
        else
            current += ch
        end
    end
    
    if current.size > 0
        result.push_back(current)
    end
    
    return result
end
```

### 使用自定义包

```covscript
# main.csc
import string_utils

var text = "Hello World"
var reversed = string_utils.reverse(text)
system.out.println("Reversed: " + reversed)

var upper = string_utils.toUpperCase(text)
system.out.println("Upper: " + upper)

var parts = string_utils.split("one,two,three", ',')
foreach part in parts
    system.out.println(part)
end
```

## 2.7.7 模块搜索路径

CovScript 在以下位置搜索模块：

1. 当前工作目录
2. 环境变量 `CS_IMPORT_PATH` 指定的路径
3. 标准库路径（由 `CS_DEV_PATH` 指定）

```covscript
# 查看导入路径
system.out.println("Import paths:")
foreach path in runtime.get_import_path().split({system.path.delimiter})
    system.out.println("  " + path)
end
```

## 2.7.8 实际应用示例

### 日志记录包

```covscript
# logger.csp
package logger
    var logLevel = "INFO"
    
    function setLevel(level)
        logLevel = level
    end
    
    function log(level, message)
        var timestamp = runtime.time()
        system.out.println("[" + to_string(timestamp) + "] [" + level + "] " + message)
    end
    
    function info(message)
        log("INFO", message)
    end
    
    function warning(message)
        log("WARNING", message)
    end
    
    function error(message)
        log("ERROR", message)
    end
    
    function debug(message)
        if logLevel == "DEBUG"
            log("DEBUG", message)
        end
    end
end
```

### 配置管理包

```covscript
# config.csp
package config
    var settings = new hash_map
    
    function load(filename)
        var file = iostream.fstream(filename, iostream.openmode.in)
        loop
            var line = file.getline()
            if file.eof()
                break
            end
            
            # 解析配置行（简化版）
            var parts = line.split("=")
            if parts.size == 2
                settings.insert(parts[0], parts[1])
            end
        end
        file.close()
    end
    
    function get(key, defaultValue)
        if settings.exist(key)
            return settings.at(key)
        end
        return defaultValue
    end
    
    function set(key, value)
        settings.insert(key, value)
    end
    
    function save(filename)
        var file = iostream.fstream(filename, iostream.openmode.out)
        foreach item in settings
            file.println(item.first + "=" + item.second)
        end
        file.close()
    end
end
```

### 使用包构建应用

```covscript
# app.csc
import logger
import config

# 初始化日志
logger.setLevel("DEBUG")
logger.info("Application starting...")

# 加载配置
config.load("app.conf")
var appName = config.get("app_name", "MyApp")
var port = config.get("port", "8080")

logger.info("App name: " + appName)
logger.info("Port: " + port)

# 应用逻辑
logger.debug("Processing data...")
# ...

logger.info("Application finished")
```

## 模块系统最佳实践

1. **使用包组织相关功能**：将相关的函数、类放在同一个包中
2. **避免循环依赖**：模块之间不应该相互依赖
3. **使用有意义的包名**：包名应该清楚地表达其功能
4. **文档化公共接口**：为包的公共函数和类添加注释
5. **版本管理**：在包中包含版本信息

```covscript
# good_package.csp
package utils
    # 版本信息
    constant VERSION = "1.0.0"
    
    # 文档化的函数
    # 计算两点之间的距离
    # @param x1 第一个点的x坐标
    # @param y1 第一个点的y坐标
    # @param x2 第二个点的x坐标
    # @param y2 第二个点的y坐标
    # @return 距离值
    function distance(x1, y1, x2, y2)
        var dx = x2 - x1
        var dy = y2 - y1
        return math.sqrt(dx * dx + dy * dy)
    end
end
```
