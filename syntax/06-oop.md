# 2.6 面向对象编程

CovScript 支持面向对象编程，提供了结构体、类、继承等特性。

## 2.6.1 结构体（struct）

结构体是一种轻量级的数据容器，用于组织相关的数据。

```covscript
# 定义结构体
struct Point
    var x = 0
    var y = 0
end

# 创建结构体实例
var p1 = new Point
p1.x = 10
p1.y = 20

system.out.println("Point: (" + to_string(p1.x) + ", " + to_string(p1.y) + ")")

# 带初始值的结构体
struct Rectangle
    var width = 0
    var height = 0
    var x = 0
    var y = 0
end

var rect = new Rectangle
rect.width = 100
rect.height = 50
rect.x = 10
rect.y = 10
```

### 结构体方法

结构体可以包含方法。

```covscript
struct Circle
    var x = 0
    var y = 0
    var radius = 1
    
    # 定义方法
    function getArea()
        return 3.14159 * this.radius * this.radius
    end
    
    function getCircumference()
        return 2 * 3.14159 * this.radius
    end
    
    function moveTo(newX, newY)
        this.x = newX
        this.y = newY
    end
end

var circle = new Circle
circle.radius = 5
system.out.println("Area: " + to_string(circle.getArea()))
system.out.println("Circumference: " + to_string(circle.getCircumference()))

circle.moveTo(100, 200)
system.out.println("New position: (" + to_string(circle.x) + ", " + to_string(circle.y) + ")")
```

## 2.6.2 类（class）

类是更强大的面向对象结构，支持构造函数、继承等特性。

```covscript
# 定义类
class Person
    var name = ""
    var age = 0
    
    function speak()
        system.out.println("My name is " + this.name)
    end
    
    function introduce()
        system.out.println("I am " + this.name + ", " + to_string(this.age) + " years old")
    end
end

# 创建类实例
var person = new Person
person.name = "Alice"
person.age = 25
person.speak()
person.introduce()
```

## 2.6.3 构造函数（construct）

**仅 ECS 支持：** 构造函数 `construct` 是 CovScript 4 (ECS) 的特性，CSC 不支持此功能。

构造函数在对象创建时自动调用，用于初始化对象。使用构造函数可以在创建对象时直接传入参数，避免创建后再逐个设置属性。

```covscript
class Student
    var name = ""
    var age = 0
    var grade = ""
    
    # 构造函数：在对象创建时自动调用
    function construct(n, a, g)
        this.name = n
        this.age = a
        this.grade = g
    end
    
    function display()
        system.out.println("Student: " + this.name)
        system.out.println("Age: " + to_string(this.age))
        system.out.println("Grade: " + this.grade)
    end
end

# 使用构造函数创建对象：new ClassName{参数列表}
var student1 = new Student{"Alice", 18, "A"}
var student2 = new Student{"Bob", 19, "B"}

student1.display()
student2.display()
```

### 默认构造函数

```covscript
class Book
    var title = "Untitled"
    var author = "Unknown"
    var pages = 0
    
    # 默认构造函数
    function initialize()
        this.title = "Default Book"
        this.author = "Default Author"
        this.pages = 100
    end
    
    function info()
        system.out.println(this.title + " by " + this.author + " (" + to_string(this.pages) + " pages)")
    end
end

var book = new Book
book.info()
```

**重要说明：** 即便定义了 `construct` 构造函数，`initialize` 默认构造函数依然会先被调用。调用顺序为：
1. 先调用 `initialize()` 进行默认初始化
2. 再调用 `construct(参数)` 进行参数化初始化

这个设计确保了对象的基本状态总是被正确初始化

## 2.6.4 继承（extends）

类可以继承另一个类的属性和方法。

```covscript
# 基类
class Animal
    var name = ""
    var age = 0
    
    function construct(n, a)
        this.name = n
        this.age = a
    end
    
    function makeSound()
        system.out.println("Some sound...")
    end
    
    function info()
        system.out.println("Name: " + this.name + ", Age: " + to_string(this.age))
    end
end

# 派生类
class Dog extends Animal
    var breed = ""
    
    function construct(n, a, b) override
        # 调用父类构造函数（通过手动设置）
        this.name = n
        this.age = a
        this.breed = b
    end
    
    function makeSound() override
        system.out.println("Woof! Woof!")
    end
    
    function fetch() override
        system.out.println(this.name + " is fetching the ball!")
    end
end

class Cat extends Animal
    var color = ""
    
    function construct(n, a, c) override
        this.name = n
        this.age = a
        this.color = c
    end
    
    function makeSound() override
        system.out.println("Meow! Meow!")
    end
    
    function climb() override
        system.out.println(this.name + " is climbing the tree!")
    end
end

# 使用继承
var dog = new Dog{"Buddy", 3, "Golden Retriever"}
dog.info()
dog.makeSound()
dog.fetch()

var cat = new Cat{"Whiskers", 2, "Orange"}
cat.info()
cat.makeSound()
cat.climb()
```

## 2.6.5 方法重写（override）

子类可以重写父类的方法。

```covscript
class Shape
    var color = "white"
    
    function construct(c)
        this.color = c
    end
    
    function draw()
        system.out.println("Drawing a shape")
    end
    
    function getArea()
        return 0
    end
end

class Square extends Shape
    var side = 0
    
    function construct(c, s) override
        this.color = c
        this.side = s
    end
    
    # 重写父类方法
    function draw() override
        system.out.println("Drawing a " + this.color + " square")
    end
    
    function getArea() override
        return this.side * this.side
    end
end

class Circle extends Shape
    var radius = 0
    
    function construct(c, r) override
        this.color = c
        this.radius = r
    end
    
    # 重写父类方法
    function draw() override
        system.out.println("Drawing a " + this.color + " circle")
    end
    
    function getArea() override
        return 3.14159 * this.radius * this.radius
    end
end

# 多态
var shapes = {new Square{"red", 5}, new Circle{"blue", 3}}

foreach shape in shapes
    shape.draw()
    system.out.println("Area: " + to_string(shape.getArea()))
end
```

## 2.6.6 this 关键字

`this` 关键字引用当前对象实例。

```covscript
class Counter
    var count = 0
    
    function increment()
        this.count += 1
        return this  # 返回当前对象，支持链式调用
    end
    
    function decrement()
        this.count -= 1
        return this
    end
    
    function reset()
        this.count = 0
        return this
    end
    
    function getValue()
        return this.count
    end
end

# 使用 this 实现链式调用
var counter = new Counter
counter.increment().increment().increment()
system.out.println(counter.getValue())  # 3

counter.reset().increment()
system.out.println(counter.getValue())  # 1
```

## 2.6.7 指针操作

### 引用操作符（&）

获取对象的引用。

```covscript
class Data
    var value = 0
    
    function construct(v)
        this.value = v
    end
end

var obj = new Data{42}

# 获取引用
var ref = &obj

# 通过引用修改
ref.value = 100
system.out.println(obj.value)  # 100
```

### 箭头操作符（->）

访问指针指向对象的成员。

```covscript
class Node
    var data = 0
    var next = null
    
    function construct(d)
        this.data = d
        this.next = null
    end
end

var node1 = new Node{1}
var node2 = new Node{2}

node1.next = node2

# 访问对象成员（使用点运算符）
var nextNode = node1.next
system.out.println(nextNode.data)  # 2

# 链式访问
if node1.next != null
    system.out.println(node1.next.data)
end

# 如果使用 gcnew 创建指针，则使用箭头运算符
var ptrNode = gcnew Node{3}
system.out.println(ptrNode->data)  # 3
```

### 解引用操作符（*）

获取指针指向的值。

```covscript
var x = 42
var ptr = gcnew x

# 解引用
var value = *ptr
system.out.println(value)  # 42

# 通过指针修改值
*ptr = 100
system.out.println(*ptr)  # 100
```

## 2.6.8 完整示例：链表实现

```covscript
class Node
    var data = null
    var next = null
    
    function construct(d)
        this.data = d
        this.next = null
    end
end

class LinkedList
    var head = null
    var size = 0
    
    function construct()
        this.head = null
        this.size = 0
    end
    
    function append(data)
        var newNode = gcnew Node{data}
        
        if this.head == null
            this.head = newNode
        else
            var current = this.head
            loop
                if current->next == null
                    current->next = newNode
                    break
                end
                current = current->next
            end
        end
        
        this.size += 1
    end
    
    function display()
        if this.head == null
            system.out.println("List is empty")
            return
        end
        
        var current = this.head
        var output = ""
        
        loop
            output += to_string(current->data)
            if current->next != null
                output += " -> "
                current = current->next
            else
                break
            end
        end
        
        system.out.println(output)
    end
    
    function getSize()
        return this.size
    end
end

# 使用链表
var lst = new LinkedList
lst.append(1)
lst.append(2)
lst.append(3)
lst.append(4)

system.out.println("List size: " + to_string(lst.getSize()))
lst.display()
```

## 面向对象编程最佳实践

1. **封装**：隐藏内部实现细节，只暴露必要的接口
2. **单一职责**：每个类应该只有一个职责
3. **组合优于继承**：优先使用组合而不是继承来复用代码
4. **使用有意义的命名**：类名、方法名应该清晰表达其用途
5. **保持类的简洁**：避免过大的类，适时拆分

```covscript
# 好的设计示例
class BankAccount
    var balance = 0
    var accountNumber = ""
    
    function construct(accNum, initialBalance)
        this.accountNumber = accNum
        this.balance = initialBalance
    end
    
    function deposit(amount)
        if amount > 0
            this.balance += amount
            return true
        end
        return false
    end
    
    function withdraw(amount)
        if amount > 0 && amount <= this.balance
            this.balance -= amount
            return true
        end
        return false
    end
    
    function getBalance()
        return this.balance
    end
end
```
