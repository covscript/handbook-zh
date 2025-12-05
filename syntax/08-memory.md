# 2.8 内存管理

CovScript 提供了灵活的内存管理机制，包括自动内存管理和手动内存控制。

## 2.8.1 new 关键字

`new` 关键字用于创建对象实例。

```covscript
# 创建基本对象
var list = new list
var map = new hash_map
var set = new hash_set

# 创建自定义类实例
class Person
    var name = ""
    var age = 0
    
    function construct(n, a)
        this.name = n
        this.age = a
    end
end

var person = new Person("Alice", 25)

# 创建结构体实例
struct Point
    var x = 0
    var y = 0
end

var point = new Point
point.x = 10
point.y = 20
```

### 对象生命周期

CovScript 使用引用计数进行自动内存管理。当对象不再被引用时，会自动释放。

```covscript
function createList()
    var list = new list
    list.push_back(1)
    list.push_back(2)
    list.push_back(3)
    return list
end

# list 在函数返回后继续存在
var myList = createList()

# 当 myList 超出作用域或被重新赋值时，对象会被自动释放
myList = null  # 显式释放引用
```

## 2.8.2 指针（pointer）

### gcnew - 垃圾回收指针

`gcnew` 创建由垃圾回收器管理的指针。

```covscript
# 创建 GC 管理的指针
var x = 42
var ptr = gcnew x

# 解引用指针
var value = *ptr
system.out.println(value)  # 42

# 修改指针指向的值
*ptr = 100
system.out.println(*ptr)  # 100

# 指针会被自动管理，无需手动释放
```

### 指针与对象

```covscript
class Data
    var value = 0
    
    function construct(v)
        this.value = v
    end
    
    function display()
        system.out.println("Value: " + to_string(this.value))
    end
end

# 创建对象指针
var obj = new Data(42)
var objPtr = gcnew obj

# 通过指针访问对象
(*objPtr).display()

# 或使用箭头操作符
objPtr->display()
```

## 2.8.3 指针操作

### 解引用操作符（*）

获取指针指向的值。

```covscript
var num = 100
var ptr = gcnew num

# 读取值
var value = *ptr
system.out.println("Value: " + to_string(value))

# 修改值
*ptr = 200
system.out.println("New value: " + to_string(*ptr))
```

### 箭头操作符（->）

访问指针指向对象的成员。

```covscript
class Rectangle
    var width = 0
    var height = 0
    
    function construct(w, h)
        this.width = w
        this.height = h
    end
    
    function getArea()
        return this.width * this.height
    end
end

var rect = new Rectangle(10, 5)
var rectPtr = rect  # 对象本身就是引用

# 使用箭头访问成员
var area = rectPtr->getArea()
system.out.println("Area: " + to_string(area))

# 修改成员
rectPtr->width = 20
system.out.println("New area: " + to_string(rectPtr->getArea()))
```

### 取址操作符（&）

获取变量的引用。

```covscript
var x = 10
var ref = &x

# 修改引用会影响原变量
ref = 20
system.out.println(x)  # 20

# 函数参数传递引用
function increment(ref)
    ref += 1
end

var count = 5
increment(&count)
system.out.println(count)  # 6
```

## 2.8.4 move 语义

`move` 语义用于转移对象的所有权，避免不必要的拷贝。

```covscript
# 移动对象
var list1 = new list
list1.push_back(1)
list1.push_back(2)
list1.push_back(3)

# 移动 list1 的内容到 list2
var list2 = move(list1)

# list1 现在为空或无效
# list2 拥有原来的数据
system.out.println("List2 size: " + to_string(list2.size()))  # 3
```

### 移动语义在函数中的应用

```covscript
function createLargeList()
    var list = new list
    for var i = 0; i < 1000; ++i
        list.push_back(i)
    end
    return move(list)  # 移动而不是拷贝
end

var data = createLargeList()
system.out.println("Size: " + to_string(data.size()))
```

### 移动与交换

```covscript
# 使用 move 实现高效的 swap
function swap(a, b)
    var temp = move(a)
    a = move(b)
    b = move(temp)
end

var x = {1, 2, 3}
var y = {4, 5, 6}

swap(&x, &y)

# 现在 x 包含 {4, 5, 6}
# y 包含 {1, 2, 3}
```

## 2.8.5 引用与值的区别

### 值传递

```covscript
function modifyValue(x)
    x = 100
end

var num = 10
modifyValue(num)
system.out.println(num)  # 10（未改变）
```

### 引用传递

```covscript
function modifyReference(ref)
    ref = 100
end

var num = 10
modifyReference(&num)
system.out.println(num)  # 100（已改变）
```

### 对象传递（默认为引用）

```covscript
class Counter
    var count = 0
    
    function increment()
        this.count += 1
    end
end

function incrementCounter(counter)
    counter.increment()
end

var c = new Counter
c.count = 5
incrementCounter(c)
system.out.println(c.count)  # 6（已改变）
```

## 2.8.6 内存管理实践

### 避免内存泄漏

```covscript
# 好的实践：及时释放不需要的引用
var data = new hash_map
# 使用 data...

# 完成后释放
data = null

# 循环引用可能导致内存问题
class Node
    var value = 0
    var next = null
    
    function construct(v)
        this.value = v
    end
end

# 注意清理循环引用
var node1 = new Node(1)
var node2 = new Node(2)
node1.next = node2
node2.next = node1  # 循环引用

# 清理
node1.next = null
node2.next = null
```

### 资源管理

```covscript
# 使用 RAII 模式管理资源
class FileHandler
    var file = null
    
    function construct(filename)
        this.file = iostream.fstream(filename, iostream.openmode.in)
    end
    
    function read()
        if this.file != null
            return this.file.getline()
        end
        return null
    end
    
    function close()
        if this.file != null
            this.file.close()
            this.file = null
        end
    end
end

# 使用
var handler = new FileHandler("data.txt")
var line = handler.read()
handler.close()
```

## 2.8.7 完整示例：智能指针模拟

```covscript
class SmartPointer
    var ptr = null
    var refCount = null
    
    function construct(obj)
        this.ptr = obj
        this.refCount = gcnew 1
    end
    
    function get()
        return this.ptr
    end
    
    function addRef()
        if this.refCount != null
            *this.refCount += 1
        end
    end
    
    function release()
        if this.refCount != null
            *this.refCount -= 1
            if *this.refCount == 0
                this.ptr = null
                this.refCount = null
                system.out.println("Object released")
            end
        end
    end
    
    function getRefCount()
        if this.refCount != null
            return *this.refCount
        end
        return 0
    end
end

# 使用智能指针
var obj = new hash_map
var sp1 = new SmartPointer(obj)

system.out.println("Ref count: " + to_string(sp1.getRefCount()))  # 1

var sp2 = sp1
sp2.addRef()
system.out.println("Ref count: " + to_string(sp1.getRefCount()))  # 2

sp2.release()
system.out.println("Ref count: " + to_string(sp1.getRefCount()))  # 1

sp1.release()
# 输出：Object released
```

## 2.8.8 内存性能优化

### 预分配容器

```covscript
# 预分配列表容量（如果支持）
var list = new list
# list.reserve(1000)  # 预留空间

for var i = 0; i < 1000; ++i
    list.push_back(i)
end
```

### 复用对象

```covscript
# 对象池模式
class ObjectPool
    var available = new list
    var inUse = new list
    
    function construct()
        # 预创建一些对象
        for var i = 0; i < 10; ++i
            this.available.push_back(new MyObject())
        end
    end
    
    function acquire()
        if this.available.size() > 0
            var obj = this.available.back()
            this.available.pop_back()
            this.inUse.push_back(obj)
            return obj
        end
        
        # 创建新对象
        var obj = new MyObject()
        this.inUse.push_back(obj)
        return obj
    end
    
    function release(obj)
        # 从 inUse 移除并放回 available
        # 简化实现
        this.available.push_back(obj)
    end
end
```

## 内存管理最佳实践

1. **使用自动内存管理**：优先依赖引用计数，避免手动管理
2. **及时释放大对象**：不再需要时置为 `null`
3. **避免循环引用**：注意对象之间的引用关系
4. **使用 move 优化性能**：大对象转移所有权时使用 move
5. **注意作用域**：合理使用局部变量减少内存占用

```covscript
# 好的实践
function processData(filename)
    var data = loadLargeFile(filename)
    var result = transformData(data)
    data = null  # 及时释放
    return result
end

# 避免在循环中创建大对象
var cache = new hash_map
for var i = 0; i < 1000; ++i
    if !cache.exist(i)
        cache.insert(i, createExpensiveObject(i))
    end
    processWithCache(i, cache)
end
```
