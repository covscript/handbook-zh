# 2.4 控制流

控制流语句用于控制程序的执行顺序。CovScript 提供了丰富的控制流结构。

## 2.4.1 条件语句

### if 语句

基本的条件判断语句。

```covscript
var age = 18

# 简单 if 语句
if age >= 18
    system.out.println("成年人")
end

# if 语句可以包含多行代码
if age >= 18
    system.out.println("你已经成年了")
    system.out.println("可以考驾照了")
end
```

### if-else 语句

提供两个分支的条件判断。

```covscript
var score = 75

if score >= 60
    system.out.println("及格")
else
    system.out.println("不及格")
end

# 嵌套的条件判断
var temperature = 25
if temperature > 30
    system.out.println("很热")
else
    if temperature > 20
        system.out.println("温暖")
    else
        system.out.println("凉爽")
    end
end
```

### 多重条件（else if）

```covscript
var score = 85

if score >= 90
    system.out.println("优秀")
else
    if score >= 80
        system.out.println("良好")
    else
        if score >= 60
            system.out.println("及格")
        else
            system.out.println("不及格")
        end
    end
end

# 更清晰的写法（推荐）
# **仅 ECS 支持：** CSC 中需要使用嵌套的 if-else
if score >= 90
    system.out.println("优秀")
else if score >= 80
    system.out.println("良好")
else if score >= 60
    system.out.println("及格")
else
    system.out.println("不及格")
end
```

**ECS 与 CSC 区别：** 
- **ECS** 支持 `else if` 语法，可以直接链式判断多个条件
- **CSC** 需要使用嵌套的 `if-else` 结构

```covscript
# CSC 中的等效写法
if score >= 90
    system.out.println("优秀")
else
    if score >= 80
        system.out.println("良好")
    else
        if score >= 60
            system.out.println("及格")
        else
            system.out.println("不及格")
        end
    end
end
```

## 2.4.2 switch 语句

switch 语句用于多分支选择，不同 case 的类型可以不一样。

```covscript
var day = 3

switch day
    case 1
        system.out.println("星期一")
    end
    case 2
        system.out.println("星期二")
    end
    case 3
        system.out.println("星期三")
    end
    case 4
        system.out.println("星期四")
    end
    case 5
        system.out.println("星期五")
    end
    case 6
        system.out.println("星期六")
    end
    case 7
        system.out.println("星期日")
    end
    default
        system.out.println("无效的日期")
    end
end

# 字符串匹配
var command = "start"

switch command
    case "start"
        system.out.println("启动程序")
    end
    case "stop"
        system.out.println("停止程序")
    end
    case "restart"
        system.out.println("重启程序")
    end
    default
        system.out.println("未知命令")
    end
end
```

## 2.4.3 循环语句

### while 循环

当条件为真时重复执行代码块。

```covscript
# 基本 while 循环
var count = 0
while count < 5
    system.out.println("Count: " + to_string(count))
    ++count
end

# 计算累加和
var sum = 0
var i = 1
while i <= 10
    sum += i
    ++i
end
system.out.println("Sum: " + to_string(sum))  # 55
```

### loop 循环

无限循环，需要使用 break 退出。

```covscript
var counter = 0
loop
    system.out.println(counter)
    ++counter
    if counter >= 5
        break
    end
end

# 另一个例子
var running = true
loop
    # 执行某些操作
    system.out.print("继续运行？(y/n): ")
    var input = system.in.getline()
    
    if input == "n"
        break
    end
end
```

### loop-until 循环

执行代码块直到条件为真。

```covscript
var num = 0
loop
    system.out.println(num)
    ++num
until num >= 5

# 等价于
var num = 0
loop
    system.out.println(num)
    ++num
    if num >= 5
        break
    end
end
```

### for 循环

用于固定次数的循环。注意：CovScript 的 `for` 循环使用逗号分隔。

```covscript
# 基本 for 循环（使用逗号分隔）
for i=0, i<5, ++i
    system.out.println("Iteration: " + to_string(i))
end

# 步长不为1的循环
for i=0, i<10, i+=2
    system.out.println(i)  # 0, 2, 4, 6, 8
end

# 倒序循环
for i=10, i>0, --i
    system.out.println(i)
end

# 嵌套循环
for i=1, i<=3, ++i
    for j=1, j<=3, ++j
        system.out.println(to_string(i) + " * " + to_string(j) + " = " + to_string(i * j))
    end
end
```

### foreach 循环

用于遍历容器中的元素。

```covscript
# 遍历数组
var numbers = {1, 2, 3, 4, 5}
foreach num in numbers
    system.out.println(num)
end

# 遍历列表
var names = new list
names.push_back("Alice")
names.push_back("Bob")
names.push_back("Charlie")

foreach name in names
    system.out.println("Hello, " + name)
end

# 遍历哈希映射
var scores = new hash_map
scores.insert("Alice", 95)
scores.insert("Bob", 87)
scores.insert("Charlie", 92)

foreach item in scores
    system.out.println(item.first + ": " + to_string(item.second))
end

# 遍历字符串
# 字符串比较特殊，无法在 foreach 循环中修改值
var text = "Hello"
foreach ch in text
    system.out.println(ch)
end
```

## 2.4.4 循环控制

### continue 语句

跳过当前迭代，继续下一次循环。

```covscript
# 跳过偶数
for i=1, i<=10, ++i
    if i % 2 == 0
        continue
    end
    system.out.println(i)  # 只打印奇数
end

# 在 foreach 中使用
var numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
foreach num in numbers
    if num % 3 == 0
        continue  # 跳过3的倍数
    end
    system.out.println(num)
end
```

### break 语句

立即退出循环。

```covscript
# 找到第一个大于50的数
var numbers = {23, 45, 67, 12, 89, 34}
foreach num in numbers
    if num > 50
        system.out.println("找到: " + to_string(num))
        break
    end
end

# 在嵌套循环中使用（只退出内层循环）
for i=0, i<5, ++i
    for j=0, j<5, ++j
        if j == 3
            break  # 只退出内层循环
        end
        system.out.println(to_string(i) + ", " + to_string(j))
    end
end
```

### return 语句

退出函数并返回值（在函数章节详细介绍）。

```covscript
function findMax(arr)
    if arr.empty
        return null
    end
    
    var maxVal = arr[0]
    foreach val in arr
        if val > maxVal
            maxVal = val
        end
    end
    return maxVal
end

var numbers = {3, 7, 2, 9, 1, 5}
var max = findMax(numbers)
system.out.println("最大值: " + to_string(max))
```

## 控制流最佳实践

1. **避免过深的嵌套**：使用 `else if` 代替多层嵌套的 `if-else`
2. **合理使用 break 和 continue**：提高代码可读性
3. **注意无限循环**：使用 `loop` 时确保有退出条件
4. **优先使用 foreach**：遍历容器时更简洁
5. **保持代码简洁**：复杂的条件应提取为函数

```covscript
# 好的实践
var status = checkStatus()
if status == "error"
    handleError()
    return
end

# 继续正常流程
processData()

# 避免的写法
var status = checkStatus()
if status != "error"
    processData()
else
    handleError()
end
```
