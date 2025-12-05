# 2.13 异步编程与协程

CovScript 提供了强大的协程和异步编程支持，让开发者能够编写高效的并发程序。本章介绍如何使用 `fiber`（协程）模块和相关的运行时函数来实现异步操作。

## 2.13.1 协程基础

协程（Fiber）是一种轻量级的并发机制，它允许函数在执行过程中暂停并在稍后恢复。

### 创建协程

使用 `fiber.create()` 创建协程。

```covscript
import fiber

# 创建一个简单的协程
var f = fiber.create([]() {
    system.out.println("协程开始执行")
    fiber.yield()  # 暂停协程
    system.out.println("协程恢复执行")
})

# 首次执行协程
system.out.println("主程序：启动协程")
fiber.resume(f)

# 恢复协程执行
system.out.println("主程序：恢复协程")
fiber.resume(f)
system.out.println("主程序：结束")
```

输出：
```
主程序：启动协程
协程开始执行
主程序：恢复协程
协程恢复执行
主程序：结束
```

### 协程的三个核心函数

1. **fiber.create(func)** - 创建新协程
2. **fiber.yield()** - 暂停当前协程，将控制权返回给调用者
3. **fiber.resume(fiber)** - 恢复协程的执行

```covscript
import fiber

# 带参数的协程
var counter = fiber.create([](name, count) {
    for i = 0, i < count, ++i
        system.out.println(name + ": " + to_string(i))
        fiber.yield()
    end
})

# 传递参数并执行
fiber.resume(counter, "任务1", 3)
system.out.println("---")
fiber.resume(counter)
system.out.println("---")
fiber.resume(counter)
```

## 2.13.2 协程的生命周期

协程有以下几种状态：
- **创建态**：协程已创建但未启动
- **运行态**：协程正在执行
- **暂停态**：协程执行到 `fiber.yield()` 后暂停
- **结束态**：协程函数执行完毕

```covscript
import fiber

function checkStatus(f)
    if fiber.is_alive(f)
        system.out.println("协程仍在运行")
    else
        system.out.println("协程已结束")
    end
end

var task = fiber.create([]() {
    system.out.println("步骤 1")
    fiber.yield()
    system.out.println("步骤 2")
    fiber.yield()
    system.out.println("步骤 3")
})

fiber.resume(task)
checkStatus(task)  # 协程仍在运行

fiber.resume(task)
checkStatus(task)  # 协程仍在运行

fiber.resume(task)
checkStatus(task)  # 协程已结束
```

## 2.13.3 协程间通信

协程可以通过 `yield` 和 `resume` 传递数据。

```covscript
import fiber

# 生产者协程
var producer = fiber.create([]() {
    for i = 1, i <= 5, ++i
        system.out.println("生产: " + to_string(i))
        fiber.yield(i)  # 将数据传递给调用者
    end
})

# 消费者逻辑
for i = 1, i <= 5, ++i
    var value = fiber.resume(producer)
    system.out.println("消费: " + to_string(value))
end
```

### 双向通信

```covscript
import fiber

var processor = fiber.create([]() {
    loop
        var input = fiber.yield(null)  # 等待输入
        if input == null
            break
        end
        var result = input * 2
        fiber.yield(result)  # 返回结果
    end
})

# 启动协程
fiber.resume(processor)

# 发送数据并获取结果
for i = 1, i <= 3, ++i
    fiber.resume(processor, i)
    var result = fiber.resume(processor)
    system.out.println(to_string(i) + " * 2 = " + to_string(result))
end

# 结束协程
fiber.resume(processor, null)
```

## 2.13.4 异步等待（runtime.await）

`runtime.await()` 函数用于等待协程完成，并返回协程的最终结果。

```covscript
import fiber

# 异步任务
function asyncTask(taskName, duration)
    return fiber.create([](name, dur) {
        system.out.println(name + " 开始")
        runtime.sleep(dur)
        system.out.println(name + " 完成")
        return name + " 的结果"
    }, taskName, duration)
end

# 创建异步任务
var task1 = asyncTask("任务A", 1000)
var task2 = asyncTask("任务B", 2000)

# 等待任务完成
var result1 = runtime.await(task1)
system.out.println("收到: " + result1)

var result2 = runtime.await(task2)
system.out.println("收到: " + result2)
```

### 并发执行多个任务

```covscript
import fiber

function createWorker(id, workload)
    return fiber.create([](workerId, work) {
        system.out.println("工作器 " + to_string(workerId) + " 开始处理")
        var sum = 0
        for i = 0, i < work, ++i
            sum += i
            if i % 1000 == 0
                fiber.yield()  # 定期让出控制权
            end
        end
        system.out.println("工作器 " + to_string(workerId) + " 完成")
        return sum
    }, id, workload)
end

# 创建多个工作器
var workers = new list
for i = 1, i <= 5, ++i
    workers.push_back(createWorker(i, i * 10000))
end

# 并发执行所有工作器
var results = new list
foreach worker in workers
    results.push_back(runtime.await(worker))
end

# 输出结果
var index = 0
foreach result in results
    index += 1
    system.out.println("工作器 " + to_string(index) + " 结果: " + to_string(result))
end
```

## 2.13.5 超时处理

### runtime.wait_for - 等待指定时间

`runtime.wait_for()` 在指定时间内等待协程完成。如果超时，抛出异常。

```covscript
import fiber

function slowTask()
    return fiber.create([]() {
        system.out.println("开始缓慢任务...")
        runtime.sleep(5000)  # 5秒
        system.out.println("缓慢任务完成")
        return "完成"
    })
end

# 尝试在 2 秒内完成任务
var task = slowTask()
try
    var result = runtime.wait_for(task, 2000)  # 等待最多 2 秒
    system.out.println("任务成功: " + result)
catch e
    system.out.println("任务超时: " + to_string(e))
end
```

### runtime.wait_until - 等待到指定时间点

`runtime.wait_until()` 等待协程直到达到指定的时间戳。

```covscript
import fiber

function timedTask()
    return fiber.create([]() {
        system.out.println("任务开始于: " + to_string(runtime.time()))
        runtime.sleep(3000)
        system.out.println("任务完成于: " + to_string(runtime.time()))
        return "完成"
    })
end

var task = timedTask()
var deadline = runtime.time() + 5000  # 当前时间 + 5秒

try
    var result = runtime.wait_until(task, deadline)
    system.out.println("在截止时间前完成: " + result)
catch e
    system.out.println("未能在截止时间前完成: " + to_string(e))
end
```

## 2.13.6 实际应用案例

### 案例1：并发下载器

模拟并发下载多个文件。

```covscript
import fiber

# 模拟文件下载
function downloadFile(url, size)
    return fiber.create([](fileUrl, fileSize) {
        system.out.println("开始下载: " + fileUrl)
        var progress = 0
        
        loop
            if progress >= fileSize
                break
            end
            
            # 模拟下载一部分数据
            var chunk = 100
            progress += chunk
            if progress > fileSize
                progress = fileSize
            end
            
            # 显示进度
            var percent = (progress * 100) / fileSize
            system.out.println(fileUrl + ": " + to_string(percent) + "%")
            
            runtime.sleep(100)  # 模拟网络延迟
            fiber.yield()  # 让其他下载任务运行
        end
        
        system.out.println("下载完成: " + fileUrl)
        return fileUrl
    }, url, size)
end

# 创建多个下载任务
var downloads = new list
downloads.push_back(downloadFile("file1.dat", 500))
downloads.push_back(downloadFile("file2.dat", 300))
downloads.push_back(downloadFile("file3.dat", 400))

# 并发执行下载
var completed = new list
foreach download in downloads
    fiber.resume(download)  # 启动下载
end

# 循环直到所有下载完成
var allDone = false
loop
    if allDone
        break
    end
    
    allDone = true
    foreach download in downloads
        if fiber.is_alive(download)
            fiber.resume(download)
            allDone = false
        end
    end
    
    runtime.sleep(50)
end

system.out.println("所有下载已完成")
```

### 案例2：异步任务调度器

实现一个简单的任务调度系统。

```covscript
import fiber

class TaskScheduler
    var tasks = new list
    var running = false
    
    # 添加任务
    function addTask(task)
        this.tasks.push_back(task)
    end
    
    # 运行调度器
    function run()
        this.running = true
        system.out.println("调度器启动")
        
        loop
            if !this.running
                break
            end
            
            var activeTasks = new list
            
            # 执行每个任务
            foreach task in this.tasks
                if fiber.is_alive(task)
                    fiber.resume(task)
                    activeTasks.push_back(task)
                end
            end
            
            # 更新任务列表
            this.tasks = activeTasks
            
            # 如果没有活动任务，退出
            if this.tasks.size == 0
                system.out.println("没有更多任务")
                break
            end
            
            runtime.sleep(100)  # 调度间隔
        end
        
        system.out.println("调度器停止")
    end
    
    # 停止调度器
    function stop()
        this.running = false
    end
end

# 创建调度器
var scheduler = new TaskScheduler{}

# 添加任务
scheduler.addTask(fiber.create([](name) {
    for i = 1, i <= 5, ++i
        system.out.println(name + ": 步骤 " + to_string(i))
        fiber.yield()
    end
}, "任务A"))

scheduler.addTask(fiber.create([](name) {
    for i = 1, i <= 3, ++i
        system.out.println(name + ": 步骤 " + to_string(i))
        fiber.yield()
    end
}, "任务B"))

scheduler.addTask(fiber.create([](name) {
    for i = 1, i <= 4, ++i
        system.out.println(name + ": 步骤 " + to_string(i))
        fiber.yield()
    end
}, "任务C"))

# 运行调度器
scheduler.run()
```

### 案例3：生产者-消费者模式

使用协程实现经典的生产者-消费者模式。

```covscript
import fiber

class AsyncQueue
    var items = new list
    var maxSize = 10
    
    function construct(size) override
        this.maxSize = size
    end
    
    function put(item)
        loop
            if this.items.size < this.maxSize
                this.items.push_back(item)
                break
            end
            fiber.yield()  # 队列满，等待
        end
    end
    
    function get()
        loop
            if this.items.size > 0
                var item = this.items[0]
                this.items.pop_front()
                return item
            end
            fiber.yield()  # 队列空，等待
        end
    end
    
    function size()
        return this.items.size
    end
end

# 创建共享队列
var queue = new AsyncQueue{5}

# 生产者
var producer = fiber.create([](q, count) {
    for i = 1, i <= count, ++i
        system.out.println("生产: " + to_string(i))
        q.put(i)
        runtime.sleep(100)
        fiber.yield()
    end
    system.out.println("生产完成")
}, queue, 10)

# 消费者
var consumer = fiber.create([](q, count) {
    for i = 1, i <= count, ++i
        var item = q.get()
        system.out.println("消费: " + to_string(item))
        runtime.sleep(150)
        fiber.yield()
    end
    system.out.println("消费完成")
}, queue, 10)

# 交替执行生产者和消费者
fiber.resume(producer)
fiber.resume(consumer)

loop
    var producerAlive = fiber.is_alive(producer)
    var consumerAlive = fiber.is_alive(consumer)
    
    if !producerAlive && !consumerAlive
        break
    end
    
    if producerAlive
        fiber.resume(producer)
    end
    if consumerAlive
        fiber.resume(consumer)
    end
    
    runtime.sleep(50)
end

system.out.println("程序结束")
```

## 2.13.7 异步编程最佳实践

### 1. 合理使用 yield

在长时间运行的任务中定期调用 `fiber.yield()`，避免阻塞其他协程。

```covscript
import fiber

# 不好的做法：长时间计算不让出控制权
function badTask()
    return fiber.create([]() {
        var sum = 0
        for i = 0, i < 10000000, ++i
            sum += i
        end
        return sum
    })
end

# 好的做法：定期让出控制权
function goodTask()
    return fiber.create([]() {
        var sum = 0
        for i = 0, i < 10000000, ++i
            sum += i
            if i % 100000 == 0
                fiber.yield()  # 定期让出
            end
        end
        return sum
    })
end
```

### 2. 异常处理

始终在协程中处理可能的异常。

```covscript
import fiber

function safeTask(operation)
    return fiber.create([](op) {
        try
            # 执行可能失败的操作
            var result = op()
            return result
        catch e
            system.out.println("任务失败: " + to_string(e))
            return null
        end
    }, operation)
end

var task = safeTask([]() {
    # 可能抛出异常的代码
    throw "模拟错误"
})

var result = runtime.await(task)
if result != null
    system.out.println("任务成功: " + to_string(result))
else
    system.out.println("任务失败")
end
```

### 3. 避免死锁

在设计协程间通信时，确保不会出现循环等待。

```covscript
import fiber

# 使用超时避免死锁
function safeWait(task, timeout)
    try
        return runtime.wait_for(task, timeout)
    catch e
        system.out.println("等待超时，可能发生死锁")
        return null
    end
end
```

### 4. 资源清理

确保协程结束时清理资源。

```covscript
import fiber

function taskWithResource()
    return fiber.create([]() {
        var file = null
        try
            # 打开资源
            file = iostream.fstream("data.txt", iostream.openmode.in)
            
            # 处理数据
            loop
                var line = file.getline()
                if file.eof()
                    break
                end
                # 处理行
                fiber.yield()
            end
            
        catch e
            system.out.println("错误: " + to_string(e))
        finally
            # 清理资源
            if file != null
                file.close()
            end
        end
    })
end
```

## 2.13.8 性能考虑

1. **协程开销**：协程创建和切换有开销，不要创建过多协程
2. **内存使用**：每个协程都会占用一定内存，注意控制并发数量
3. **调度策略**：合理设计协程调度，避免饥饿现象
4. **CPU vs I/O**：协程更适合 I/O 密集型任务，CPU 密集型任务考虑其他方案

```covscript
import fiber

# 限制并发数量
class WorkerPool
    var maxWorkers = 5
    var activeWorkers = 0
    var taskQueue = new list
    
    function construct(max) override
        this.maxWorkers = max
    end
    
    function submit(task)
        if this.activeWorkers < this.maxWorkers
            this.activeWorkers += 1
            this.executeTask(task)
        else
            this.taskQueue.push_back(task)
        end
    end
    
    function executeTask(task)
        # 在新协程中执行任务
        fiber.create([](t, pool) {
            try
                runtime.await(t)
            finally
                pool.activeWorkers -= 1
                # 从队列中取下一个任务
                if pool.taskQueue.size > 0
                    var nextTask = pool.taskQueue[0]
                    pool.taskQueue.pop_front()
                    pool.submit(nextTask)
                end
            end
        }, task, this)
    end
end
```

## 总结

CovScript 的协程和异步编程功能提供了：

- **轻量级并发**：使用协程实现高效的并发
- **简洁的 API**：`fiber.create`、`fiber.yield`、`fiber.resume` 三个核心函数
- **灵活的控制**：可以精确控制协程的执行流程
- **超时支持**：`runtime.wait_for` 和 `runtime.wait_until` 处理超时
- **实用工具**：`runtime.await` 简化异步任务等待

通过合理使用这些特性，可以编写出高效、清晰的异步程序。
