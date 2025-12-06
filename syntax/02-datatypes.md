# 2.2 数据类型

CovScript 提供了丰富的数据类型，包括基本类型和容器类型。作为动态类型语言，变量的类型在运行时确定。

## 2.2.1 基本类型

### 数字类型 (Number)

CovScript 支持整数和浮点数。

```covscript
# 整数
var integer = 42
var negative = -10

# 浮点数
var pi = 3.14159

# 运算
var sum = 10 + 5        # 15
var product = 3.5 * 2   # 7.0
```

### 布尔类型 (Boolean)

布尔类型只有两个值：`true` 和 `false`。

```covscript
var isActive = true
var isComplete = false

# 布尔运算
var result = true && false  # false
var check = true || false   # true
var inverted = !true        # false
```

### 空值类型 (Null)

使用 `null` 表示空值或无效值。在 CovScript 中，`null` 本质上是一个空指针。

```covscript
var empty = null
var uninit = null

# 检查空值
if empty == null
    system.out.println("Value is null")
end
```

### 字符类型 (Char)

字符类型表示单个字符，使用单引号包围。

```covscript
var ch = 'A'
var digit = '5'
var symbol = '@'

# 获取字符的 ASCII 值
var ascii = to_integer(ch)
```

### 字符串类型 (String)

字符串使用双引号包围，支持多种操作。

```covscript
var str = "Hello, CovScript!"
var name = "Alice"
var multiline = "First line\nSecond line"

# 字符串拼接
var greeting = "Hello, " + name + "!"

# 字符串追加
var text = "Hello"
text.append(", World!")  # text 现在是 "Hello, World!"

# 字符串长度
var len = str.size  # 注意：size 是属性

# 字符串索引（从0开始）
var firstChar = str[0]  # 'H'
var lastChar = str[-1]  # '!'
```

## 2.2.2 容器类型

### 数组 (Array)

数组是固定大小的容器，使用 `{}` 初始化。

```covscript
# 创建数组
var arr = {1, 2, 3, 4, 5}
var names = {"Alice", "Bob", "Charlie"}

# 访问元素
var first = arr[0]      # 1
var second = arr[1]     # 2
var last = arr[-1]      # 5

# 修改元素，可以是不同类型
arr[0] = "1"
# 使用 [] 访问元素可自动扩容
arr[5] = 6

# 获取数组大小
var size = arr.size   # 6，注意 size 是属性
```

### 列表 (List)

列表是动态大小的容器，可以随时添加或删除元素。

```covscript
# 创建空列表
var list = new list

# 添加元素
list.push_back(1)
list.push_back(2)
list.push_back(3)

# 访问元素
var first = list.front
var last = list.back

# 删除元素
list.pop_back()

# 遍历列表
foreach item in list
    system.out.println(item)
end
```

### 键值对 (Pair)

键值对存储两个关联的值。

```covscript
# 创建键值对（使用冒号运算符）
var pair = 1:"one"

# 访问元素（可以使用 .first/.second 或 .key/.value）
var key = pair.first    # 或 pair.key
var value = pair.second # 或 pair.value

# 修改元素
pair.first = 2
pair.second = "two"
```

### 哈希映射表 (Hash Map)

哈希映射表用于存储键值对，提供快速查找。

```covscript
# 创建空哈希映射表
var map = new hash_map

# 添加键值对
map.insert("name", "Alice")
map.insert("age", 25)
# 访问不存在的键会自动创建
map["city"] = "Shanghai"

# 访问元素（可以使用 .at 或下标语法）
var name = map.at("name")
var age = map["age"]        # 也可以使用下标访问

# 对于字符串键，还可以使用点运算符
var city = map.city

# 检查键是否存在
if map.exist("age")
    system.out.println("Age exists")
end

# 删除键值对
map.erase("city")

# 遍历哈希映射
foreach item in map
    system.out.println(item.key + ": " + item.value)
end

# 也可以从数组快捷创建
# @begin @end 是预处理指令，用于告诉编译器这里要多行
# **ECS 与 CSC 区别：** ECS 可自动处理跨行逻辑，无需使用 @begin/@end
# 以下写法在 CSC 中需要 @begin/@end，在 ECS 中可直接换行
@begin
map = {
    "name":"Alice",
    "age":25,
    "city": "Shanghai"
}.to_hash_map()
@end
```

### 哈希集合 (Hash Set)

哈希集合用于存储唯一的元素。

```covscript
# 创建哈希集合
var set = new hash_set

# 添加元素
set.insert(1)
set.insert(2)
set.insert(3)
set.insert(1)  # 重复元素不会被添加

# 检查元素是否存在
if set.exist(2)
    system.out.println("2 is in the set")
end

# 删除元素
set.erase(3)

# 获取集合大小
var size = set.size

# 集合运算
var set1 = {1, 2, 3}.to_hash_set()

var set2 = {2, 3, 4}.to_hash_set()

# 并集
var union_set = hash_set.merge(set1, set2)      # {1, 2, 3, 4}

# 交集
var intersect_set = hash_set.intersect(set1, set2)  # {2, 3}

# 差集
var diff_set = hash_set.subtract(set1, set2)       # {1}
```

## 2.2.3 类型检查

### 使用 typeid 运算符

`typeid` 运算符用于比较类型。

```covscript
var x = 10
var y = "hello"

# 类型比较
if typeid x == typeid 0
    system.out.println("x is a number")
end

if typeid y == typeid ""
    system.out.println("y is a string")
end

# 判断两个变量类型是否相同
if typeid x == typeid y
    system.out.println("Same type")
else
    system.out.println("Different types")
end
```

### `is` 运算符

```covscript
var x = 10
var y = "hello"

if x is integer
    system.out.println("x is a integer")
end

if x not float
    system.out.println("x not a float")
end

if y is string
    system.out.println("y is a string")
end
```

**仅 ECS 支持：** `is` 运算符是 CovScript 4 (ECS) 的特性，CSC 中需要使用 `typeid` 进行类型检查。在面向对象编程中，`is` 运算符可以方便地判断继承关系

### 使用 type() 函数

`type()` 函数返回变量的类型字符串，主要用于调试输出。在实际编程中，更常用 `typeid` 进行类型检查。

```covscript
var num = 42
var str = "hello"
var arr = {1, 2, 3}

# 获取类型
var numType = type(num)
var strType = type(str)
var arrType = type(arr)

# 打印类型信息
system.out.println(numType)  # number
system.out.println(strType)  # string
system.out.println(arrType)  # array
```

### 类型转换

```covscript
# 转换为字符串
var num = 42
var str = to_string(num)     # "42"

# 转换为整数
var s = "123"
var n = to_integer(s)        # 123

# 转换为浮点数
var f = to_number("3.14")    # 3.14
```

**ECS 与 CSC 区别：** ECS 提供了更简洁的 `as` 类型转换语法

```covscript
# 仅 ECS 支持的 as 语法
# 转换为字符串
var num = 42
var str = num as string     # "42"

# 转换为整数
var s = "123"
var n = s as integer        # 123

# 转换为浮点数
var f = "3.14" as number   # 3.14
```

**说明：** `as` 运算符提供了更自然的类型转换语法，但仅在 ECS 中可用。CSC 中必须使用 `to_string()`、`to_integer()` 等函数

## 类型系统注意事项

- CovScript 是动态类型语言，变量类型在运行时确定
- 变量可以在任何时候改变类型
- 类型转换可能会失败，需要适当的错误处理
- 容器类型可以包含不同类型的元素
