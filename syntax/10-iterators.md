# 2.10 迭代器

迭代器是用于遍历容器元素的对象。CovScript 提供了强大的迭代器支持。

## 2.10.1 获取迭代器

### begin 和 end

使用 `.begin` 和 `.end` 获取容器的起始和结束迭代器。

```covscript
# 列表迭代器
var list = new list
list.push_back(1)
list.push_back(2)
list.push_back(3)

var it = list.begin
var itEnd = list.end

# 遍历列表
loop
    if it == itEnd
        break
    end
    system.out.println(it.data)
    it.next()
end
```

### 数组迭代器

```covscript
var arr = {10, 20, 30, 40, 50}

var it = arr.begin
var itEnd = arr.end

loop
    if it == itEnd
        break
    end
    system.out.println("Element: " + to_string(it.data))
    it.next()
end
```

## 2.10.2 迭代器操作

### data - 获取当前元素

```covscript
var list = new list
list.push_back("Alice")
list.push_back("Bob")
list.push_back("Charlie")

var it = list.begin
var name = it.data  # 获取当前元素
system.out.println(name)  # Alice
```

### next() - 移动到下一个元素

```covscript
var numbers = {1, 2, 3, 4, 5}
var it = numbers.begin

# 获取第一个元素
system.out.println(it.data)  # 1

# 移动到下一个
it.next()
system.out.println(it.data)  # 2

# 再移动
it.next()
system.out.println(it.data)  # 3
```

### 比较迭代器

```covscript
var list = new list
list.push_back(10)
list.push_back(20)

var it1 = list.begin
var it2 = list.begin
var itEnd = list.end

# 比较迭代器
if it1 == it2
    system.out.println("Same position")
end

if it1 != itEnd
    system.out.println("Not at end")
end
```

## 2.10.3 迭代器方法

### 正向遍历

```covscript
function printList(list)
    var it = list.begin
    var itEnd = list.end
    
    system.out.print("[")
    var first = true
    
    loop
        if it == itEnd
            break
        end
        
        if !first
            system.out.print(", ")
        end
        first = false
        
        system.out.print(to_string(it.data))
        it.next()
    end
    
    system.out.println("]")
end

var numbers = new list
numbers.push_back(1)
numbers.push_back(2)
numbers.push_back(3)

printList(numbers)  # [1, 2, 3]
```

### 查找元素

```covscript
function findElement(list, target)
    var it = list.begin
    var itEnd = list.end
    var index = 0
    
    loop
        if it == itEnd
            return -1  # 未找到
        end
        
        if it.data == target
            return index  # 返回索引
        end
        
        it.next()
        index += 1
    end
end

var names = new list
names.push_back("Alice")
names.push_back("Bob")
names.push_back("Charlie")

var pos = findElement(names, "Bob")
system.out.println("Position: " + to_string(pos))  # 1
```

### 修改元素

```covscript
# 通过迭代器修改元素（对于支持的容器）
var list = new list
list.push_back(1)
list.push_back(2)
list.push_back(3)

# 遍历并修改
var it = list.begin
var itEnd = list.end

loop
    if it == itEnd
        break
    end
    
    # 修改元素（需要通过其他方式）
    var value = it.data
    # 对于某些容器，可能需要重新赋值
    
    it.next()
end
```

## 2.10.4 容器特定的迭代器

### 哈希映射迭代器

```covscript
var map = new hash_map
map.insert("name", "Alice")
map.insert("age", 25)
map.insert("city", "Shanghai")

# 遍历哈希映射
var it = map.begin
var itEnd = map.end

loop
    if it == itEnd
        break
    end
    
    var pair = it.data
    system.out.println(pair.first + " = " + to_string(pair.second))
    
    it.next()
end
```

### 哈希集合迭代器

```covscript
var set = new hash_set
set.insert(10)
set.insert(20)
set.insert(30)
set.insert(20)  # 重复元素不会添加

# 遍历集合
var it = set.begin
var itEnd = set.end

loop
    if it == itEnd
        break
    end
    
    system.out.println("Element: " + to_string(it.data))
    it.next()
end
```

## 2.10.5 迭代器与 foreach

`foreach` 是迭代器的语法糖，内部使用迭代器实现。

```covscript
var list = new list
list.push_back(1)
list.push_back(2)
list.push_back(3)

# 使用 foreach（推荐）
foreach item in lst
    system.out.println(item)
end

# 等价的迭代器写法
var it = lst.begin
var itEnd = lst.end
loop
    if it == itEnd
        break
    end
    var item = it.data
    system.out.println(item)
    it.next()
end
```

## 2.10.6 迭代器实用函数

### 计数

```covscript
function count(container)
    var it = container.begin
    var itEnd = container.end
    var cnt = 0
    
    loop
        if it == itEnd
            break
        end
        cnt += 1
        it.next()
    end
    
    return cnt
end

var list = new list
list.push_back(1)
list.push_back(2)
list.push_back(3)

system.out.println("Count: " + to_string(count(list)))
```

### 求和

```covscript
function sum(container)
    var it = container.begin
    var itEnd = container.end
    var total = 0
    
    loop
        if it == itEnd
            break
        end
        total += it.data
        it.next()
    end
    
    return total
end

var numbers = {1, 2, 3, 4, 5}
system.out.println("Sum: " + to_string(sum(numbers)))  # 15
```

### 过滤

```covscript
function filter(container, predicate)
    var result = new list
    var it = container.begin
    var itEnd = container.end
    
    loop
        if it == itEnd
            break
        end
        
        var item = it.data
        if predicate(item)
            result.push_back(item)
        end
        
        it.next()
    end
    
    return result
end

var numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
var evens = filter(numbers, [](x) -> x % 2 == 0)

foreach num in evens
    system.out.println(num)
end
```

### 映射

```covscript
function map(container, mapper)
    var result = new list
    var it = container.begin
    var itEnd = container.end
    
    loop
        if it == itEnd
            break
        end
        
        var item = it.data
        result.push_back(mapper(item))
        
        it.next()
    end
    
    return result
end

var numbers = {1, 2, 3, 4, 5}
var squared = map(numbers, [](x) -> x * x)

foreach num in squared
    system.out.println(num)
end
```

### 折叠/归约

```covscript
function reduce(container, initial, reducer)
    var it = container.begin
    var itEnd = container.end
    var accumulator = initial
    
    loop
        if it == itEnd
            break
        end
        
        accumulator = reducer(accumulator, it.data)
        it.next()
    end
    
    return accumulator
end

var numbers = {1, 2, 3, 4, 5}

# 求和
var sum = reduce(numbers, 0, [](acc, x) -> acc + x)
system.out.println("Sum: " + to_string(sum))  # 15

# 求积
var product = reduce(numbers, 1, [](acc, x) -> acc * x)
system.out.println("Product: " + to_string(product))  # 120
```

## 2.10.7 迭代器与标准容器

CovScript 的标准容器（list, array, hash_map, hash_set 等）都内置了迭代器支持。

```covscript
# 所有标准容器都支持迭代器
var containers = new list

# 列表
var myList = new list
myList.push_back(1)
myList.push_back(2)
containers.push_back(myList)

# 数组
var myArray = {10, 20, 30}
containers.push_back(myArray)

# 哈希映射
var myMap = new hash_map
myMap.insert("key", "value")
containers.push_back(myMap)

# 遍历所有容器
foreach container in containers
    system.out.println("Container elements:")
    var it = container.begin
    var itEnd = container.end
    
    loop
        if it == itEnd
            break
        end
        
        system.out.println("  " + to_string(it.data))
        it.next()
    end
end
```

## 2.10.8 迭代器性能考虑

### 避免频繁复制

```covscript
# 低效：每次都复制迭代器
function inefficientSum(lst)
    var total = 0
    for it = lst.begin, it != lst.end, it.next()
        total += it.data
    end
    return total
end

# 高效：减少迭代器操作
function efficientSum(lst)
    var total = 0
    foreach item in lst
        total += item
    end
    return total
end
```

### 缓存 end 迭代器

```covscript
# 好的实践：缓存结束迭代器
var lst = new list
lst.push_back(1)
lst.push_back(2)
lst.push_back(3)

var it = lst.begin
var itEnd = lst.end  # 缓存

loop
    if it == itEnd
        break
    end
    system.out.println(it.data)
    it.next()
end
```

## 迭代器最佳实践

1. **优先使用 foreach**：更简洁和易读
2. **缓存 end 迭代器**：避免重复访问 `.end`
3. **注意迭代器失效**：修改容器可能导致迭代器失效
4. **使用合适的容器**：根据访问模式选择容器类型
5. **避免不必要的复制**：迭代器操作可能涉及复制

```covscript
# 推荐的迭代方式
var lst = new list
lst.push_back(1)
lst.push_back(2)
lst.push_back(3)

# 简单遍历：使用 foreach
foreach item in lst
    system.out.println(item)
end

# 需要索引或修改：使用传统循环
for i = 0, i < lst.size, ++i
    var item = lst[i]
    # 处理 item
end

# 需要精确控制：使用迭代器
var it = lst.begin
var itEnd = lst.end
loop
    if it == itEnd
        break
    end
    # 精确控制迭代过程
    it.next()
end
```
