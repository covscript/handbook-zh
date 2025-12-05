# 2.12 高级特性

CovScript 提供了一系列高级特性，让开发更加灵活和高效。

## 2.12.1 do 表达式

`do` 表达式允许在表达式位置执行语句块并返回值。

```covscript
# 基本 do 表达式
var result = do
    var x = 10
    var y = 20
    x + y  # 最后一个表达式作为返回值
end

system.out.println(result)  # 30

# 在函数调用中使用
system.out.println(do
    var msg = "Hello"
    msg + ", World!"
end)

# 复杂的 do 表达式
var value = do
    var sum = 0
    for i=1,i <= 10,++i
        sum += i
    end
    sum
end

system.out.println("Sum: " + to_string(value))
```

### do 表达式的实际应用

```covscript
# 条件初始化
var config = do
    if system.path.exist("config.json")
        loadConfig("config.json")
    else
        getDefaultConfig()
    end
end

# 复杂计算
var statistics = do
    var data = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
    var sum = 0
    var count = 0
    
    foreach num in data
        sum += num
        count += 1
    end
    
    var mean = sum / count
    new hash_map.insert("sum", sum).insert("count", count).insert("mean", mean)
end
```

## 2.12.2 范围函数（range）

`range` 函数生成数字序列。

```covscript
# 基本范围
foreach i in range(5)
    system.out.println(i)  # 0, 1, 2, 3, 4
end

# 指定起始和结束
foreach i in range(1, 6)
    system.out.println(i)  # 1, 2, 3, 4, 5
end

# 指定步长
foreach i in range(0, 10, 2)
    system.out.println(i)  # 0, 2, 4, 6, 8
end

# 倒序范围
foreach i in range(10, 0, -1)
    system.out.println(i)  # 10, 9, 8, ..., 1
end
```

### range 的实际应用

```covscript
# 生成序列
var numbers = new list
foreach i in range(1, 11)
    numbers.push_back(i)
end

# 批量操作
foreach i in range(100)
    var filename = "file_" + to_string(i) + ".txt"
    # 处理文件
end

# 矩阵初始化
var matrix = new list
foreach i in range(5)
    var row = new list
    foreach j in range(5)
        row.push_back(0)
    end
    matrix.push_back(row)
end
```

## 2.12.3 链式调用

方法返回 `this` 实现链式调用。

```covscript
class StringBuilder
    var buffer = ""
    
    function append(text)
        this.buffer += text
        return this
    end
    
    function appendLine(text)
        this.buffer += text + "\n"
        return this
    end
    
    function clear()
        this.buffer = ""
        return this
    end
    
    function toString()
        return this.buffer
    end
end

# 链式调用
var sb = new StringBuilder
var result = sb.append("Hello")
               .append(" ")
               .append("World")
               .appendLine("!")
               .append("How are you?")
               .toString()

system.out.println(result)
```

### 流式API

```covscript
class Query
    var data = null
    var filtered = null
    
    function construct(arr)
        this.data = arr
        this.filtered = arr
    end
    
    function where(predicate)
        var result = new list
        foreach item in this.filtered
            if predicate(item)
                result.push_back(item)
            end
        end
        this.filtered = result
        return this
    end
    
    function select(mapper)
        var result = new list
        foreach item in this.filtered
            result.push_back(mapper(item))
        end
        this.filtered = result
        return this
    end
    
    function toList()
        return this.filtered
    end
end

# 使用流式API
var numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
var result = new Query(numbers)
    .where([](x) -> x % 2 == 0)      # 筛选偶数
    .select([](x) -> x * x)           # 平方
    .toList()

foreach num in result
    system.out.println(num)  # 4, 16, 36, 64, 100
end
```

## 2.12.4 初始化列表

使用 `{}` 语法快速初始化容器。

```covscript
# 数组初始化
var numbers = {1, 2, 3, 4, 5}
var names = {"Alice", "Bob", "Charlie"}
var mixed = {1, "two", 3.0, true}

# 嵌套初始化
var matrix = {
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9}
}

# 访问嵌套元素
system.out.println(matrix[0][0])  # 1
system.out.println(matrix[1][2])  # 6

# 空数组
var empty = {}
```

### 结构化数据

```covscript
# 配置对象
var config = {
    "host": "localhost",
    "port": 8080,
    "timeout": 30,
    "retries": 3
}

# 记录列表
var users = {
    {"name": "Alice", "age": 25},
    {"name": "Bob", "age": 30},
    {"name": "Charlie", "age": 35}
}
```

## 2.12.5 克隆（clone）

复制对象，创建独立副本。

```covscript
# 克隆数组
var original = {1, 2, 3, 4, 5}
var copy = clone(original)

copy[0] = 100
system.out.println(original[0])  # 1（原始未改变）
system.out.println(copy[0])      # 100

# 克隆列表
var list1 = new list
list1.push_back(1)
list1.push_back(2)
list1.push_back(3)

var list2 = clone(list1)
list2.push_back(4)

system.out.println(list1.size)  # 3
system.out.println(list2.size)  # 4

# 克隆对象
class Point
    var x = 0
    var y = 0
end

var p1 = new Point
p1.x = 10
p1.y = 20

var p2 = clone(p1)
p2.x = 30

system.out.println(p1.x)  # 10
system.out.println(p2.x)  # 30
```

### 深拷贝与浅拷贝

```covscript
# 浅拷贝（引用相同的对象）
var data1 = {
    "name": "Alice",
    "scores": {90, 85, 95}
}

var data2 = data1  # 浅拷贝，共享引用
data2["name"] = "Bob"
# 修改会影响原始数据

# 深拷贝（完全独立的副本）
function deepClone(obj)
    # 实现深拷贝逻辑
    if typeid obj == typeid {}
        var result = {}
        foreach item in obj
            result.push_back(deepClone(item))
        end
        return result
    else
        return clone(obj)
    end
end
```

## 2.12.6 类型转换（as）

使用 `as` 进行类型转换（如果支持）。

```covscript
# 安全类型转换
var value = "123"
var num = value as number  # 转换为数字

# 对象类型转换
class Base
end

class Derived extends Base
end

var obj = new Derived
var base = obj as Base  # 向上转换
```

## 2.12.7 操作符重载

为自定义类型重载操作符（根据语言支持）。

```covscript
class Vector
    var x = 0
    var y = 0
    
    function construct(a, b)
        this.x = a
        this.y = b
    end
    
    # 重载加法（概念示例）
    function add(other)
        return new Vector(this.x + other.x, this.y + other.y)
    end
    
    # 重载乘法
    function multiply(scalar)
        return new Vector(this.x * scalar, this.y * scalar)
    end
    
    function toString()
        return "(" + to_string(this.x) + ", " + to_string(this.y) + ")"
    end
end

var v1 = new Vector(1, 2)
var v2 = new Vector(3, 4)
var v3 = v1.add(v2)
var v4 = v1.multiply(2)

system.out.println(v3.toString())  # (4, 6)
system.out.println(v4.toString())  # (2, 4)
```

## 2.12.8 元编程

使用反射和动态特性实现元编程。

```covscript
# 动态访问属性
function getProperty(obj, name)
    # 根据名称获取属性
    # 实现取决于语言支持
end

function setProperty(obj, name, value)
    # 根据名称设置属性
end

# 动态方法调用
function callMethod(obj, methodName, args)
    # 动态调用方法
end

# 类型信息
function getTypeInfo(obj)
    return {
        "type": type(obj),
        "typeid": typeid obj,
        "string": to_string(obj)
    }
end
```

## 2.12.9 模式匹配（概念）

使用 switch 和条件实现模式匹配。

```covscript
function processValue(value)
    # 类型匹配
    if typeid value == typeid 0
        system.out.println("Number: " + to_string(value))
    else if typeid value == typeid ""
        system.out.println("String: " + value)
    else if typeid value == typeid {}
        system.out.println("Array with " + to_string(value.size) + " elements")
    else
        system.out.println("Unknown type")
    end
end

# 值匹配
function processCommand(cmd)
    switch cmd
        case "start"
            startService()
        end
        case "stop"
            stopService()
        end
        case "restart"
            stopService()
            startService()
        end
        default
            system.out.println("Unknown command")
        end
    end
end
```

## 2.12.10 装饰器模式

使用高阶函数实现装饰器。

```covscript
# 计时装饰器
function timed(func)
    return [](...args) {
        var start = runtime.time()
        var result = func(...args)
        var end = runtime.time()
        system.out.println("Execution time: " + to_string(end - start) + "ms")
        return result
    }
end

# 日志装饰器
function logged(func)
    return [](...args) {
        system.out.println("Calling function with args: " + to_string(args))
        var result = func(...args)
        system.out.println("Function returned: " + to_string(result))
        return result
    }
end

# 使用装饰器
function slowFunction(n)
    runtime.sleep(1000)
    return n * n
end

var decoratedFunc = timed(logged(slowFunction))
var result = decoratedFunc(5)
```

## 2.12.11 惰性求值

实现延迟计算。

```covscript
class Lazy
    var computer = null
    var value = null
    var computed = false
    
    function construct(func)
        this.computer = func
        this.computed = false
    end
    
    function get()
        if !this.computed
            this.value = this.computer()
            this.computed = true
        end
        return this.value
    end
end

# 使用惰性求值
var expensive = new Lazy([]() {
    system.out.println("Computing expensive value...")
    runtime.sleep(2000)
    return 42
})

system.out.println("Created lazy value")
# 这里不会计算

system.out.println("Getting value...")
var result = expensive.get()  # 第一次调用时计算
system.out.println("Result: " + to_string(result))

var result2 = expensive.get()  # 使用缓存的值
system.out.println("Result2: " + to_string(result2))
```

## 2.12.12 管道操作

实现函数组合和数据管道。

```covscript
class Pipeline
    var value = null
    
    function construct(v)
        this.value = v
    end
    
    function pipe(func)
        this.value = func(this.value)
        return this
    end
    
    function get()
        return this.value
    end
end

# 使用管道
var result = new Pipeline(10)
    .pipe([](x) -> x * 2)      # 20
    .pipe([](x) -> x + 5)      # 25
    .pipe([](x) -> x * x)      # 625
    .get()

system.out.println("Result: " + to_string(result))

# 字符串处理管道
function uppercase(str)
    # 转大写实现
    return str
end

function trim(str)
    # 去空格实现
    return str
end

function reverse(str)
    # 反转实现
    var result = ""
    for i=str.size - 1,i >= 0,--i
        result += str[i]
    end
    return result
end

var processed = new Pipeline("  hello world  ")
    .pipe(trim)
    .pipe(uppercase)
    .pipe(reverse)
    .get()
```

## 高级特性最佳实践

1. **合理使用 do 表达式**：简化复杂的初始化逻辑
2. **链式调用提高可读性**：设计流畅的API
3. **注意克隆的性能开销**：大对象克隆成本高
4. **元编程需谨慎**：过度使用会降低代码可维护性
5. **装饰器模式分离关注点**：增强功能而不修改原始代码

```covscript
# 综合示例：构建器模式
class QueryBuilder
    var table = ""
    var columns = new list
    var whereClause = ""
    var orderBy = ""
    var limit = -1
    
    function from(tableName)
        this.table = tableName
        return this
    end
    
    function select(cols)
        this.columns = cols
        return this
    end
    
    function where(condition)
        this.whereClause = condition
        return this
    end
    
    function order(field)
        this.orderBy = field
        return this
    end
    
    function limitTo(n)
        this.limit = n
        return this
    end
    
    function build()
        var sql = "SELECT "
        
        if this.columns.size == 0
            sql += "*"
        else
            sql += join(this.columns, ", ")
        end
        
        sql += " FROM " + this.table
        
        if !this.whereClause.empty()
            sql += " WHERE " + this.whereClause
        end
        
        if !this.orderBy.empty()
            sql += " ORDER BY " + this.orderBy
        end
        
        if this.limit > 0
            sql += " LIMIT " + to_string(this.limit)
        end
        
        return sql
    end
end

# 使用构建器
var query = new QueryBuilder()
    .from("users")
    .select({"id", "name", "email"})
    .where("age > 18")
    .order("name")
    .limitTo(10)
    .build()

system.out.println(query)
```
