# 2.5 函数

函数是组织代码的基本单元，用于封装可重用的代码块。CovScript 支持多种函数定义方式。

## 2.5.1 函数定义

使用 `function` 关键字定义函数。

```covscript
# 基本函数定义
function greet(name)
    system.out.println("Hello, " + name + "!")
end

# 调用函数
greet("Alice")
greet("Bob")

# 带返回值的函数
function add(a, b)
    return a + b
end

var result = add(10, 20)
system.out.println(result)  # 30
```

## 2.5.2 无参数函数

函数可以不接受任何参数。

```covscript
# 无参数函数
function sayHello()
    system.out.println("Hello, World!")
end

# 调用
sayHello()

# 返回当前时间的函数
function getCurrentTime()
    return runtime.time()
end

var time = getCurrentTime()
system.out.println("Current time: " + to_string(time))
```

## 2.5.3 多参数函数

函数可以接受多个参数。

```covscript
# 多参数函数
function calculateArea(width, height)
    return width * height
end

var area = calculateArea(10, 5)
system.out.println("Area: " + to_string(area))

# 更多参数
function printInfo(name, age, city, country)
    system.out.println("Name: " + name)
    system.out.println("Age: " + to_string(age))
    system.out.println("City: " + city)
    system.out.println("Country: " + country)
end

printInfo("Alice", 25, "Shanghai", "China")
```

## 2.5.4 可变参数

使用 `...` 语法定义可变参数函数。

```covscript
# 可变参数函数
function sum(...args)
    var total = 0
    foreach arg in args
        total += arg
    end
    return total
end

# 调用可变参数函数
system.out.println(sum(1, 2, 3))           # 6
system.out.println(sum(1, 2, 3, 4, 5))     # 15
system.out.println(sum(10, 20))            # 30

# 可变参数和固定参数混合
function printWithPrefix(prefix, ...items)
    foreach item in items
        system.out.println(prefix + to_string(item))
    end
end

printWithPrefix("Item: ", 1, 2, 3, 4)
```

## 2.5.5 Lambda 表达式

Lambda 表达式是匿名函数，用于简洁地定义简单函数。

```covscript
# 基本 Lambda 表达式
var square = [](x) -> x * x
system.out.println(square(5))  # 25

# 多参数 Lambda
var add = [](a, b) -> a + b
system.out.println(add(10, 20))  # 30

# 作为参数传递
function applyOperation(x, y, operation)
    return operation(x, y)
end

var result1 = applyOperation(10, 5, [](a, b) -> a + b)  # 15
var result2 = applyOperation(10, 5, [](a, b) -> a * b)  # 50

# 捕获外部变量
var factor = 10
var multiply = [](x) -> x * factor
system.out.println(multiply(5))  # 50
```

### Lambda 表达式的高级用法

```covscript
# 数组操作
var numbers = {1, 2, 3, 4, 5}
var doubled = new list

foreach num in numbers
    doubled.push_back([](x) -> x * 2(num))
end

# 过滤操作
function filter(arr, predicate)
    var result = new list
    foreach item in arr
        if predicate(item)
            result.push_back(item)
        end
    end
    return result
end

var numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
var evens = filter(numbers, [](x) -> x % 2 == 0)

# 映射操作
function map(arr, mapper)
    var result = new list
    foreach item in arr
        result.push_back(mapper(item))
    end
    return result
end

var squared = map(numbers, [](x) -> x * x)
```

## 2.5.6 函数参数类型标注

CovScript 支持为函数参数添加类型标注（主要用于文档和可读性）。

```covscript
# 带类型标注的函数（注释形式）
function calculateDiscount(price, discountRate)
    # price: number - 商品原价
    # discountRate: number - 折扣率 (0-1)
    # returns: number - 折扣后价格
    
    return price * (1 - discountRate)
end

var finalPrice = calculateDiscount(100, 0.2)  # 80
```

## 2.5.7 返回值

### 单一返回值

```covscript
# 返回数字
function double(x)
    return x * 2
end

# 返回字符串
function getGreeting(name)
    return "Hello, " + name + "!"
end

# 返回布尔值
function isEven(num)
    return num % 2 == 0
end

# 提前返回
function findFirst(arr, target)
    foreach item in arr
        if item == target
            return true
        end
    end
    return false
end
```

### 多返回值

CovScript 可以通过返回容器来实现多返回值。

```covscript
# 使用数组返回多个值
function getDimensions()
    var width = 100
    var height = 200
    return {width, height}
end

var dims = getDimensions()
var w = dims[0]
var h = dims[1]
system.out.println("Width: " + to_string(w) + ", Height: " + to_string(h))

# 使用 pair 返回两个值
function divideWithRemainder(dividend, divisor)
    var quotient = dividend / divisor
    var remainder = dividend % divisor
    return new pair(quotient, remainder)
end

var result = divideWithRemainder(17, 5)
system.out.println("Quotient: " + to_string(result.first))
system.out.println("Remainder: " + to_string(result.second))

# 使用哈希映射返回命名的多个值
function getUserInfo()
    var info = new hash_map
    info.insert("name", "Alice")
    info.insert("age", 25)
    info.insert("city", "Shanghai")
    return info
end

var user = getUserInfo()
system.out.println(user.at("name"))
```

### 无返回值函数

```covscript
# 无 return 语句的函数
function printMessage(msg)
    system.out.println(msg)
    # 没有 return 语句，自动返回 null
end

var result = printMessage("Hello")  # result 为 null

# 显式返回 null
function doNothing()
    return null
end
```

## 2.5.8 递归函数

函数可以调用自身实现递归。

```covscript
# 计算阶乘
function factorial(n)
    if n <= 1
        return 1
    end
    return n * factorial(n - 1)
end

system.out.println(factorial(5))  # 120

# 计算斐波那契数列
function fibonacci(n)
    if n <= 1
        return n
    end
    return fibonacci(n - 1) + fibonacci(n - 2)
end

system.out.println(fibonacci(10))  # 55

# 递归遍历目录（示例）
function listFiles(path, indent)
    var files = system.path.scan(path)
    foreach file in files
        system.out.println(indent + file)
        if system.path.is_dir(path + "/" + file)
            listFiles(path + "/" + file, indent + "  ")
        end
    end
end
```

## 2.5.9 函数作为值

在 CovScript 中，函数是一等公民，可以作为值传递。

```covscript
# 函数赋值给变量
function add(a, b)
    return a + b
end

var operation = add
var result = operation(10, 20)  # 30

# 函数作为参数
function apply(func, x, y)
    return func(x, y)
end

function multiply(a, b)
    return a * b
end

var result1 = apply(add, 5, 3)       # 8
var result2 = apply(multiply, 5, 3)  # 15

# 返回函数
function makeMultiplier(factor)
    return [](x) -> x * factor
end

var double = makeMultiplier(2)
var triple = makeMultiplier(3)

system.out.println(double(10))  # 20
system.out.println(triple(10))  # 30
```

## 函数最佳实践

1. **单一职责**：每个函数应该只做一件事
2. **有意义的命名**：函数名应该清楚地表达其功能
3. **适当的参数数量**：避免过多参数，考虑使用对象封装
4. **避免副作用**：尽可能使函数纯粹（不修改外部状态）
5. **合理使用递归**：注意递归深度，避免栈溢出

```covscript
# 好的实践
function calculateTotalPrice(items, taxRate)
    var subtotal = 0
    foreach item in items
        subtotal += item.price * item.quantity
    end
    var tax = subtotal * taxRate
    return subtotal + tax
end

# 分离关注点
function calculateSubtotal(items)
    var total = 0
    foreach item in items
        total += item.price * item.quantity
    end
    return total
end

function calculateTax(amount, rate)
    return amount * rate
end

function getTotalWithTax(items, taxRate)
    var subtotal = calculateSubtotal(items)
    var tax = calculateTax(subtotal, taxRate)
    return subtotal + tax
end
```
