# 3.4 Extended CovScript (ECS / CovScript 4)

ECS（Extended CovScript）是 CovScript 的下一代版本，也称为 CovScript 4。ECS 是一个编译器/转译器，将扩展语法的 CovScript 代码编译为标准的 CovScript 3 代码运行。

## 特性概述

ECS 相比 CovScript 3 提供了许多新特性：

- **类型注解**：可选的类型标注
- **引用参数**：使用 `=` 前缀声明引用参数
- **箭头操作符**：`->` 用于访问成员
- **扩展导入语法**：更灵活的模块导入方式
- **改进的 Lambda 表达式**：行为更符合预期
- **独立的异常系统**：与 CovScript 3 分离的异常处理
- **构造函数分离**：支持 `new` 和 `gcnew` 带参数

## 安装

### 1. 安装 CovScript 3 运行时

首先需要安装 CovScript 3 作为基础运行时。访问 [CovScript 官网](http://covscript.org.cn) 并按照说明安装。

### 2. 通过 CSPKG 安装 ECS

```bash
cspkg install ecs_bootstrap --yes
```

安装完成后，如果使用的是 CovScript 3.4.1+ 版本，`ecs` 命令即可使用。

### 3. （可选）克隆 ECS 仓库

```bash
git clone https://github.com/covscript/ecs
```

## 使用方式

`ecs` 命令是一个编译器，将 ECS 代码转译为 CovScript 3 代码，但可以像解释器一样使用（会在运行前自动编译）。

### 基本用法

```bash
# 运行 ECS 文件
ecs program.ecs

# 运行并传递参数
ecs program.ecs arg1 arg2 arg3

# 编译到指定输出文件
ecs -o output.csc input.ecs

# 仅检查语法
ecs -c program.ecs

# 禁用编译缓存
ecs -f program.ecs
```

### 命令行选项

```
用法:
    ecs [选项...] <文件> [参数...]
    ecs [选项...]

解释器选项:
   -f            禁用编译缓存
   -m            禁用代码美化
   -c            仅检查语法
   -g            生成 cSYM 调试信息
   -d            运行调试器
   -o <路径>     设置输出路径
   -- <参数>     传递参数给 CovScript

REPL 选项:
   -s            关闭命令提示符
   -r <参数...>  设置 REPL 参数

通用选项:
   -h            显示帮助信息
   -v            显示版本信息
   -u <字符集>   设置 Unicode 字符集
                 字符集 = {"AUTO", "UTF8", "GBK"}
   -i <路径>     添加导入路径
```

## ECS 扩展语法

### 类型注解

ECS 支持可选的类型标注，增强代码可读性：

```covscript
# 函数参数类型注解
function calculate(x: number, y: number): number
    return x + y
end

# Lambda 表达式类型注解
var multiply = [](a: number, b: number): number {
    return a * b
}

# 变量类型注解
var count: number = 0
var name: string = "CovScript"
```

**注意**：类型注解仅用于文档和编辑器提示，不进行运行时类型检查。

### 引用参数

使用 `=` 前缀声明引用参数，允许函数修改传入的参数：

```covscript
# 引用参数示例
function swap(=a, =b)
    var temp = a
    a = b
    b = temp
end

var x = 10
var y = 20
swap(x, y)
system.out.println(x)  # 输出: 20
system.out.println(y)  # 输出: 10

# Lambda 中的引用参数
var increment = [](=value: number) {
    ++value
}

var count = 5
increment(count)
system.out.println(count)  # 输出: 6
```

### 箭头操作符

使用 `->` 访问对象成员：

```covscript
class Person
    var name = ""
    var age = 0
    
    function greet()
        system.out.println("Hello, I'm " + this.name)
    end
end

var person = gcnew Person
person->name = "Alice"
person->age = 25
person->greet()

# 链式调用
var result = object->method1()->method2()->value
```

### 扩展导入语法

ECS 提供更灵活的导入语法：

```covscript
# 导入并重命名
import network.tcp.socket as tcp_socket

# 导入多个模块
import parsergen,
       codec.json,
       covscript.*

# 使用别名
import database.mysql as mysql
var conn = new mysql.connection()
```

### 改进的 Lambda 表达式

ECS 中的 Lambda 表达式行为更加一致和直观：

```covscript
# 捕获外部变量
var base = 10
var add_to_base = [base](x: number): number {
    return base + x
}

system.out.println(add_to_base(5))  # 输出: 15

# 多行 Lambda
var complex_operation = [](data: array): number {
    var sum = 0
    foreach item in data
        sum += item
    end
    return sum
}
```

### 构造函数分离

在 ECS 中，需要定义独立的 `construct` 函数来支持带参数的对象构造：

```covscript
class Rectangle
    var width = 0
    var height = 0
    
    # 必须定义独立的 construct 函数
    function construct(w, h)
        this.width = w
        this.height = h
    end
    
    function area()
        return this.width * this.height
    end
end

# 使用 new 或 gcnew 创建对象
var rect1 = new Rectangle(10, 20)
var rect2 = gcnew Rectangle(30, 40)

system.out.println(rect1.area())  # 输出: 200
```

## 兼容性说明

### 1. 依赖 `ecs` 包

ECS 编译生成的程序依赖 `ecs` 包，确保目标环境已安装：

```bash
cspkg install ecs_bootstrap --yes
```

### 2. Lambda 表达式行为差异

ECS 中的 Lambda 表达式行为与 CovScript 3 有显著不同，主要体现在变量捕获和作用域方面。

### 3. 异常处理

ECS 有独立的异常系统。在 CovScript 3 中捕获 ECS 包抛出的异常时，需要转换：

```covscript
try
    # 调用 ECS 编写的函数
    some_ecs_function()
catch e
    # 转换异常以便在 CovScript 3 中处理
    e = ecs.handle_exception(e)
    system.out.println("错误: " + e.what())
end
```

### 4. 文件扩展名

- **ECS 源文件**：`.ecs`
- **编译后文件**：`.csc`（CovScript 3 格式）
- **缓存文件**：`.csym`（调试符号）

## 实践示例

### 示例 1：类型安全的数学库

```covscript
# math_utils.ecs
@charset: utf8

namespace MathUtils
    # 计算阶乘
    function factorial(n: number): number
        if n <= 1
            return 1
        end
        return n * factorial(n - 1)
    end
    
    # 计算平均值
    function average(numbers: array): number
        if numbers.size == 0
            return 0
        end
        
        var sum = 0
        foreach num in numbers
            sum += num
        end
        return sum / numbers.size
    end
    
    # 查找最大值
    function max(=result: number, numbers: array)
        if numbers.size == 0
            result = 0
            return
        end
        
        result = numbers[0]
        foreach num in numbers
            if num > result
                result = num
            end
        end
    end
end

# 使用示例
var fact_5 = MathUtils.factorial(5)
system.out.println("5! = " + to_string(fact_5))

var nums = {10, 20, 30, 40, 50}
var avg = MathUtils.average(nums)
system.out.println("平均值: " + to_string(avg))

var max_value = 0
MathUtils.max(max_value, nums)
system.out.println("最大值: " + to_string(max_value))
```

### 示例 2：面向对象编程

```covscript
# person.ecs
class Person
    var name: string = ""
    var age: number = 0
    var email: string = ""
    
    function construct(n: string, a: number, e: string)
        this.name = n
        this.age = a
        this.email = e
    end
    
    function introduce(): string
        return "我叫 " + this.name + "，今年 " + to_string(this.age) + " 岁"
    end
    
    function is_adult(): boolean
        return this.age >= 18
    end
end

class Student extends Person
    var student_id: string = ""
    var grade: number = 0
    
    function construct(n: string, a: number, e: string, sid: string, g: number)
        # 调用父类构造函数
        this.name = n
        this.age = a
        this.email = e
        this.student_id = sid
        this.grade = g
    end
    
    function introduce(): string
        return "学生 " + this.name + "，学号 " + this.student_id
    end
end

# 使用
var person = gcnew Person("张三", 25, "zhangsan@example.com")
system.out.println(person->introduce())
system.out.println("成年: " + to_string(person->is_adult()))

var student = gcnew Student("李四", 20, "lisi@example.com", "S20240001", 2)
system.out.println(student->introduce())
```

### 示例 3：函数式编程

```covscript
# functional.ecs

# 高阶函数：map
function map(func, arr: array): array
    var result = new array
    foreach item in arr
        result.push_back(func(item))
    end
    return result
end

# 高阶函数：filter
function filter(predicate, arr: array): array
    var result = new array
    foreach item in arr
        if predicate(item)
            result.push_back(item)
        end
    end
    return result
end

# 高阶函数：reduce
function reduce(func, arr: array, initial)
    var accumulator = initial
    foreach item in arr
        accumulator = func(accumulator, item)
    end
    return accumulator
end

# 使用示例
var numbers = {1, 2, 3, 4, 5}

# 每个数字加倍
var doubled = map([](x: number): number { return x * 2 }, numbers)
system.out.println("加倍: " + to_string(doubled))

# 过滤偶数
var evens = filter([](x: number): boolean { return x % 2 == 0 }, numbers)
system.out.println("偶数: " + to_string(evens))

# 求和
var sum = reduce([](acc: number, x: number): number { return acc + x }, numbers, 0)
system.out.println("总和: " + to_string(sum))
```

## 调试 ECS 程序

### 生成调试信息

```bash
# 生成 cSYM 调试符号
ecs -g program.ecs

# 运行调试器
ecs -d program.ecs
```

### 查看编译结果

```bash
# 编译并查看生成的 CSC 代码
ecs -o output.csc program.ecs
cat output.csc
```

### 禁用缓存调试

```bash
# 每次都重新编译，便于调试
ecs -f program.ecs
```

## 最佳实践

### 1. 使用类型注解增强可读性

```covscript
# 好的做法
function calculate_total(items: array, tax_rate: number): number
    var subtotal = 0
    foreach item in items
        subtotal += item
    end
    return subtotal * (1 + tax_rate)
end

# 避免
function calculate_total(items, tax_rate)
    # 不清楚参数类型
    ...
end
```

### 2. 合理使用引用参数

```covscript
# 需要修改参数时使用引用
function increment(=value: number)
    ++value
end

# 不需要修改时使用普通参数
function square(value: number): number
    return value * value
end
```

### 3. 利用箭头操作符简化代码

```covscript
# 使用箭头操作符
var result = object->method1()->method2()->value

# 相比传统方式更清晰
var temp1 = object.method1()
var temp2 = temp1.method2()
var result = temp2.value
```

### 4. 分离 ECS 和 CSC 代码

如果项目同时使用 ECS 和 CovScript 3，建议：
- 使用不同的文件夹存放 `.ecs` 和 `.csc` 文件
- ECS 代码编译后的输出单独管理
- 注意异常处理的兼容性

## 从 CovScript 3 迁移到 ECS

### 基本兼容性

大部分 CovScript 3 代码可以直接在 ECS 中运行，但有以下注意事项：

1. **构造函数**：如果使用 `new` 或 `gcnew` 传递参数，需要定义 `construct` 函数
2. **Lambda 表达式**：行为可能不同，需要测试
3. **异常处理**：捕获异常时可能需要 `ecs.handle_exception()`

### 迁移步骤

1. 将 `.csc` 文件重命名为 `.ecs`
2. 添加类型注解（可选但推荐）
3. 测试并修复不兼容的部分
4. 使用 `ecs -c` 检查语法
5. 运行测试确保功能正确

## 总结

ECS（Extended CovScript）是 CovScript 的演进版本，提供了：

- **更强的类型支持**：可选的类型注解
- **更灵活的参数传递**：引用参数
- **更清晰的语法**：箭头操作符、扩展导入
- **更好的工具链**：编译器、调试器、缓存

ECS 编译为 CovScript 3 运行，保证了向后兼容性的同时，为开发者提供了更现代化的编程体验。

**学习资源**：
- ECS GitHub: https://github.com/covscript/ecs
- CovScript 官网: http://covscript.org.cn
- 官方文档: https://manual.covscript.org.cn/
