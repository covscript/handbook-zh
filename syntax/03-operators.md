# 2.3 运算符

CovScript 提供了丰富的运算符，用于执行各种操作。

## 2.3.1 算术运算符

### 基本算术运算

```covscript
var a = 10
var b = 3

# 加法
var sum = a + b          # 13

# 减法
var diff = a - b         # 7

# 乘法
var product = a * b      # 30

# 除法
var quotient = a / b     # 3.333...

# 取模
var remainder = a % b    # 1

# 幂运算
var power = a ^ 2        # 100
```

### 自增自减运算符

```covscript
var count = 5

# 后置自增
var old = count++        # old = 5, count = 6

# 后置自减
var val = count--        # val = 6, count = 5

# 前置自增
var new = ++count        # new = 6, count = 6

# 前置自减
var current = --count    # current = 5, count = 5
```

## 2.3.2 比较运算符

比较运算符用于比较两个值，返回布尔类型结果。

```covscript
var x = 10
var y = 20

# 等于
var isEqual = (x == y)           # false

# 不等于
var notEqual = (x != y)          # true

# 大于
var greater = (x > y)            # false

# 小于
var less = (x < y)               # true

# 大于等于
var greaterOrEqual = (x >= 10)   # true

# 小于等于
var lessOrEqual = (x <= 20)      # true
```

## 2.3.3 逻辑运算符

逻辑运算符用于布尔逻辑运算。

### 运算符重载

CovScript 支持为自定义类型重载运算符。结构体可以定义 `+`, `-`, `*`, `/`, `==`, `!=` 等运算符的行为。

```covscript
struct Vector
    var x = 0
    var y = 0
    
    # 重载加法运算符
    function operator+(other)
        return new Vector {x: this.x + other.x, y: this.y + other.y}
    end
    
    # 重载乘法运算符（标量乘法）
    function operator*(scalar)
        return new Vector {x: this.x * scalar, y: this.y * scalar}
    end
end

var v1 = new Vector {x: 1, y: 2}
var v2 = new Vector {x: 3, y: 4}
var v3 = v1 + v2  # 调用重载的 + 运算符
```

### 逻辑与（AND）

```covscript
var a = true
var b = false

# 使用 && 符号
var result1 = a && b     # false

# 使用 and 关键字
var result2 = a and b    # false

# 短路求值
var x = true && (10 > 5)  # true
```

### 逻辑或（OR）

```covscript
var a = true
var b = false

# 使用 || 符号
var result1 = a || b     # true

# 使用 or 关键字
var result2 = a or b     # true

# 短路求值
var x = true || (10 / 0)  # true，不会执行除零操作
```

### 逻辑非（NOT）

```covscript
var flag = true

# 使用 ! 符号
var inverted1 = !flag    # false

# 使用 not 关键字
var inverted2 = not flag # false
```

### 复合逻辑表达式

```covscript
var age = 25
var hasLicense = true

# 复合条件
var canDrive = (age >= 18) && hasLicense

# 多重条件
var isAdult = (age >= 18) && (age < 65) && hasLicense
```

## 2.3.4 赋值运算符

### 基本赋值

```covscript
var x = 10
x = 20              # 简单赋值
```

### 复合赋值运算符

```covscript
var num = 10

# 加法赋值
num += 5            # num = num + 5, 结果为 15

# 减法赋值
num -= 3            # num = num - 3, 结果为 12

# 乘法赋值
num *= 2            # num = num * 2, 结果为 24

# 除法赋值
num /= 4            # num = num / 4, 结果为 6

# 取模赋值
num %= 4            # num = num % 4, 结果为 2
```

## 2.3.5 特殊运算符

### 点运算符（.）

点运算符用于访问命名空间、包的成员，以及对象的属性和方法。

```covscript
# 访问命名空间成员
system.out.println("Hello")

# 访问静态成员
math.abs(-5)

# 链式访问
system.console.terminal.clear()

# 访问哈希映射（字符串键）
var map = new hash_map
map.insert("name", "Alice")
var name = map.name  # 等价于 map["name"]
```

### 冒号运算符（:）

冒号运算符用于创建键值对。

```covscript
# 创建键值对
var pair = "key":"value"
var numPair = 1:100

# 在数组中使用
var pairs = {1:"one", 2:"two", 3:"three"}
```

### 箭头运算符（->）

箭头运算符用于访问指针指向的对象的成员。注意：在 CovScript 中，`new` 创建的是普通对象（引用），`gcnew` 创建的才是指针。

```covscript
# 使用 gcnew 创建指针
struct MyClass
    var value = 0
    function method()
        system.out.println("Method called")
    end
end

var ptr = gcnew MyClass
ptr->value = 10
ptr->method()
```

### 解引用运算符（*）

解引用运算符用于获取指针指向的值。使用 `gcnew` 创建指针。

```covscript
# 使用 gcnew 创建指针
var ptr = gcnew number
*ptr = 42

# 解引用读取值
var value = *ptr  # 42

# 修改指针指向的值
*ptr = 100
```

### 取址运算符（&）

取址运算符用于获取变量的引用（指针）。

```covscript
var x = 10

# 获取引用（指针）
var ptr = &x

# 通过指针修改值
*ptr = 20
system.out.println(x)  # 20
```

### 索引运算符（[]）

索引运算符用于访问数组、列表等容器的元素。

```covscript
# 数组索引
var arr = {1, 2, 3, 4, 5}
var first = arr[0]      # 1
var third = arr[2]      # 3

# 字符串索引
var str = "Hello"
var ch = str[1]         # 'e'

# 列表索引
var list = new list
list.push_back(10)
list.push_back(20)
var elem = list[0]      # 10

# 修改元素
arr[0] = 100
```

## 运算符优先级

从高到低的优先级顺序：

1. 括号 `()`
2. 成员访问 `.`, `->`
3. 一元运算符 `!`, `not`, `++`, `--`, `*`, `&`
4. 幂运算 `^`
5. 乘除模 `*`, `/`, `%`
6. 加减 `+`, `-`
7. 比较运算符 `<`, `>`, `<=`, `>=`
8. 相等运算符 `==`, `!=`
9. 逻辑与 `&&`, `and`
10. 逻辑或 `||`, `or`
11. 赋值运算符 `=`, `+=`, `-=`, 等

```covscript
# 使用括号改变优先级
var result1 = 2 + 3 * 4      # 14 (乘法优先)
var result2 = (2 + 3) * 4    # 20 (括号优先)

# 复杂表达式
var value = 10 + 5 * 2 - 3   # 10 + 10 - 3 = 17
```
