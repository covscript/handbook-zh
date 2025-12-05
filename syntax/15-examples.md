# 2.15 实战案例

本章通过实际的代码示例展示 CovScript 的应用场景，包括网络应用、游戏开发、数据库操作和文件处理等。这些案例来源于 CovScript 示例库和实际项目。

## 2.15.1 网络应用

### 案例1：简单的 HTTP 服务器

实现一个基础的 HTTP 服务器，能够处理 GET 请求并返回静态内容。

```covscript
import network

class SimpleHTTPServer
    var server = null
    var port = 8080
    var running = false
    
    function construct(p) override
        this.port = p
    end
    
    function start()
        this.server = network.tcp.server()
        this.server.bind("0.0.0.0", this.port)
        this.server.listen(10)
        this.running = true
        
        system.out.println("HTTP 服务器启动在端口 " + to_string(this.port))
        
        loop
            if !this.running
                break
            end
            
            try
                var client = this.server.accept()
                this.handleClient(client)
            catch e
                system.out.println("错误: " + to_string(e))
            end
        end
    end
    
    function handleClient(client)
        try
            # 读取请求
            var request = client.recv(4096)
            system.out.println("收到请求:")
            system.out.println(request)
            
            # 解析请求行
            var lines = this.splitLines(request)
            if lines.size > 0
                var requestLine = lines[0]
                var parts = this.splitString(requestLine, " ")
                
                if parts.size >= 2
                    var method = parts[0]
                    var path = parts[1]
                    
                    # 处理请求
                    if method == "GET"
                        this.handleGET(client, path)
                    else
                        this.send405(client)
                    end
                end
            end
            
        catch e
            system.out.println("处理请求错误: " + to_string(e))
        finally
            client.close()
        end
    end
    
    function handleGET(client, path)
        if path == "/" || path == "/index.html"
            # 返回首页
            var html = "<!DOCTYPE html><html><head><meta charset='utf-8'><title>CovScript 服务器</title></head><body><h1>欢迎使用 CovScript HTTP 服务器</h1><p>这是一个用 CovScript 编写的简单 HTTP 服务器。</p></body></html>"
            
            this.sendResponse(client, 200, "OK", "text/html; charset=utf-8", html)
        else if path == "/api/hello"
            # API 端点
            var json = "{\"message\": \"Hello from CovScript!\", \"timestamp\": " + to_string(runtime.time()) + "}"
            this.sendResponse(client, 200, "OK", "application/json", json)
        else
            # 404
            this.send404(client)
        end
    end
    
    function sendResponse(client, code, status, contentType, body)
        var response = "HTTP/1.1 " + to_string(code) + " " + status + "\r\n"
        response += "Content-Type: " + contentType + "\r\n"
        response += "Content-Length: " + to_string(body.size) + "\r\n"
        response += "Connection: close\r\n"
        response += "\r\n"
        response += body
        
        client.send(response)
    end
    
    function send404(client)
        var html = "<!DOCTYPE html><html><head><title>404 Not Found</title></head><body><h1>404 Not Found</h1></body></html>"
        this.sendResponse(client, 404, "Not Found", "text/html", html)
    end
    
    function send405(client)
        var html = "<!DOCTYPE html><html><head><title>405 Method Not Allowed</title></head><body><h1>405 Method Not Allowed</h1></body></html>"
        this.sendResponse(client, 405, "Method Not Allowed", "text/html", html)
    end
    
    function splitLines(text)
        var result = new list
        var current = ""
        
        for i = 0, i < text.size, ++i
            var ch = text[i]
            if ch == '\n'
                result.push_back(current)
                current = ""
            else if ch != '\r'
                current += ch
            end
        end
        
        if current.size > 0
            result.push_back(current)
        end
        
        return result
    end
    
    function splitString(text, delimiter)
        var result = new list
        var current = ""
        
        foreach ch in text
            if ch == delimiter
                if current.size > 0
                    result.push_back(current)
                    current = ""
                end
            else
                current += ch
            end
        end
        
        if current.size > 0
            result.push_back(current)
        end
        
        return result
    end
    
    function stop()
        this.running = false
        if this.server != null
            this.server.close()
        end
    end
end

# 启动服务器
var server = new SimpleHTTPServer{8080}
server.start()
```

### 案例2：TCP 聊天室

实现一个多客户端的聊天室应用。

```covscript
import network
import fiber

class ChatRoom
    var server = null
    var clients = new list
    var clientNames = new hash_map
    var running = false
    
    function start(port)
        this.server = network.tcp.server()
        this.server.bind("0.0.0.0", port)
        this.server.listen(10)
        this.running = true
        
        system.out.println("聊天室服务器启动在端口 " + to_string(port))
        
        loop
            if !this.running
                break
            end
            
            try
                var client = this.server.accept()
                this.clients.push_back(client)
                system.out.println("新客户端连接，当前在线: " + to_string(this.clients.size))
                
                # 为每个客户端创建处理协程
                this.createClientHandler(client)
            catch e
                system.out.println("接受连接错误: " + to_string(e))
            end
        end
    end
    
    function createClientHandler(client)
        var handler = fiber.create([](c, room) {
            var clientName = null
            
            try
                # 欢迎消息
                c.send("欢迎来到聊天室！请输入你的昵称：\n")
                
                # 获取昵称
                clientName = c.recv(1024)
                clientName = room.trim(clientName)
                room.clientNames.insert(c, clientName)
                
                # 广播加入消息
                room.broadcast(clientName + " 加入了聊天室\n", c)
                c.send("你已加入聊天室。输入消息开始聊天，输入 /quit 退出。\n")
                
                # 消息循环
                loop
                    var message = c.recv(1024)
                    if message.empty()
                        break
                    end
                    
                    message = room.trim(message)
                    
                    if message == "/quit"
                        break
                    else if message == "/list"
                        # 列出在线用户
                        c.send("在线用户 (" + to_string(room.clients.size) + "):\n")
                        foreach client in room.clients
                            if room.clientNames.exist(client)
                                c.send("  - " + room.clientNames[client] + "\n")
                            end
                        end
                    else if message.size > 0
                        # 广播消息
                        var fullMsg = clientName + ": " + message + "\n"
                        room.broadcast(fullMsg, c)
                    end
                    
                    fiber.yield()
                end
                
            catch e
                system.out.println("客户端错误: " + to_string(e))
            finally
                # 清理
                if clientName != null
                    room.broadcast(clientName + " 离开了聊天室\n", c)
                end
                
                room.removeClient(c)
                c.close()
            end
        }, client, this)
        
        fiber.resume(handler)
    end
    
    function broadcast(message, sender)
        system.out.println("[广播] " + message)
        
        foreach client in this.clients
            if client != sender
                try
                    client.send(message)
                catch e
                    # 忽略发送失败
                end
            end
        end
    end
    
    function removeClient(client)
        var newClients = new list
        foreach c in this.clients
            if c != client
                newClients.push_back(c)
            end
        end
        this.clients = newClients
        
        if this.clientNames.exist(client)
            this.clientNames.erase(client)
        end
        
        system.out.println("客户端断开，当前在线: " + to_string(this.clients.size))
    end
    
    function trim(str)
        return str.trim()
    end
    
    function stop()
        this.running = false
        
        foreach client in this.clients
            try
                client.close()
            catch e
            end
        end
        
        if this.server != null
            this.server.close()
        end
    end
end

# 启动聊天室
var chatRoom = new ChatRoom{}
chatRoom.start(9999)
```

### 聊天室客户端

```covscript
import network

class ChatClient
    var client = null
    var connected = false
    
    function connect(host, port)
        this.client = network.tcp.client()
        this.client.connect(host, port)
        this.connected = true
        
        system.out.println("已连接到服务器")
    end
    
    function run()
        # 接收欢迎消息
        var welcome = this.client.recv(1024)
        system.out.print(welcome)
        
        # 输入昵称
        var name = system.in.getline()
        this.client.send(name + "\n")
        
        # 接收确认消息
        var confirm = this.client.recv(1024)
        system.out.print(confirm)
        
        # 主循环
        loop
            if !this.connected
                break
            end
            
            # 读取用户输入
            system.out.print("> ")
            var input = system.in.getline()
            
            if input.empty()
                continue
            end
            
            # 发送消息
            this.client.send(input + "\n")
            
            if input == "/quit"
                break
            end
            
            # 接收响应
            runtime.delay(100)
            var response = this.client.recv(4096)
            if response.size > 0
                system.out.print(response)
            end
        end
    end
    
    function disconnect()
        if this.client != null
            this.client.close()
            this.connected = false
        end
    end
end

# 使用客户端
var client = new ChatClient{}
try
    client.connect("127.0.0.1", 9999)
    client.run()
catch e
    system.out.println("错误: " + to_string(e))
finally
    client.disconnect()
end
```

## 2.15.2 游戏开发

### 案例1：贪吃蛇游戏

使用控制台实现经典的贪吃蛇游戏。

```covscript
# 贪吃蛇游戏
class Point
    var x = 0
    var y = 0
    
    function construct(px, py) override
        this.x = px
        this.y = py
    end
    
    function equals(other)
        return this.x == other.x && this.y == other.y
    end
end

class Snake
    var body = new list
    var direction = "RIGHT"
    var growing = false
    
    function construct(startX, startY) override
        this.body.push_back(new Point{startX, startY})
        this.body.push_back(new Point{startX - 1, startY})
        this.body.push_back(new Point{startX - 2, startY})
    end
    
    function move()
        var head = this.body[0]
        var newHead = new Point{head.x, head.y}
        
        # 根据方向移动
        if this.direction == "UP"
            newHead.y -= 1
        else if this.direction == "DOWN"
            newHead.y += 1
        else if this.direction == "LEFT"
            newHead.x -= 1
        else if this.direction == "RIGHT"
            newHead.x += 1
        end
        
        # 添加新头部
        this.body.push_front(newHead)
        
        # 如果没有生长，移除尾部
        if !this.growing
            this.body.pop_back()
        else
            this.growing = false
        end
    end
    
    function setDirection(dir)
        # 防止反向移动
        if dir == "UP" && this.direction != "DOWN"
            this.direction = dir
        else if dir == "DOWN" && this.direction != "UP"
            this.direction = dir
        else if dir == "LEFT" && this.direction != "RIGHT"
            this.direction = dir
        else if dir == "RIGHT" && this.direction != "LEFT"
            this.direction = dir
        end
    end
    
    function grow()
        this.growing = true
    end
    
    function getHead()
        return this.body[0]
    end
    
    function checkCollision()
        var head = this.getHead()
        
        # 检查是否撞到自己
        for i = 1, i < this.body.size, ++i
            if head.equals(this.body[i])
                return true
            end
        end
        
        return false
    end
end

class SnakeGame
    var width = 40
    var height = 20
    var snake = null
    var food = null
    var score = 0
    var gameOver = false
    
    function construct() override
        this.snake = new Snake{this.width / 2, this.height / 2}
        this.spawnFood()
    end
    
    function spawnFood()
        # 简单的随机食物生成
        var x = to_integer(runtime.time() % this.width)
        var y = to_integer((runtime.time() / 100) % this.height)
        this.food = new Point{x, y}
    end
    
    function update()
        # 移动蛇
        this.snake.move()
        
        var head = this.snake.getHead()
        
        # 检查边界碰撞
        if head.x < 0 || head.x >= this.width || head.y < 0 || head.y >= this.height
            this.gameOver = true
            return
        end
        
        # 检查自身碰撞
        if this.snake.checkCollision()
            this.gameOver = true
            return
        end
        
        # 检查是否吃到食物
        if head.equals(this.food)
            this.snake.grow()
            this.score += 10
            this.spawnFood()
        end
    end
    
    function render()
        # 清屏（简化版）
        system.out.println("\n\n\n\n\n")
        
        # 绘制游戏区域
        system.out.println("贪吃蛇游戏 - 分数: " + to_string(this.score))
        system.out.println("方向键控制，Q 退出")
        
        for y = 0, y < this.height, ++y
            var line = ""
            for x = 0, x < this.width, ++x
                var point = new Point{x, y}
                var isSnake = false
                
                # 检查是否是蛇身
                foreach segment in this.snake.body
                    if segment.equals(point)
                        if segment.equals(this.snake.getHead())
                            line += "O"  # 蛇头
                        else
                            line += "o"  # 蛇身
                        end
                        isSnake = true
                        break
                    end
                end
                
                if !isSnake
                    # 检查是否是食物
                    if point.equals(this.food)
                        line += "*"
                    else
                        line += " "
                    end
                end
            end
            system.out.println("|" + line + "|")
        end
        
        # 底部边框
        var border = "+"
        for i = 0, i < this.width, ++i
            border += "-"
        end
        border += "+"
        system.out.println(border)
    end
    
    function run()
        system.out.println("游戏开始！")
        
        loop
            if this.gameOver
                break
            end
            
            # 更新游戏状态
            this.update()
            
            # 渲染
            this.render()
            
            # 简化的输入处理（实际需要非阻塞输入）
            runtime.delay(200)  # 游戏速度
            
            # 这里应该有键盘输入检测
            # 例如: var key = system.in.getKey()
            # this.snake.setDirection(...)
        end
        
        system.out.println("\n游戏结束！")
        system.out.println("最终分数: " + to_string(this.score))
    end
end

# 启动游戏
var game = new SnakeGame{}
game.run()
```

### 案例2：猜数字游戏

简单但完整的猜数字游戏。

```covscript
class GuessNumberGame
    var secretNumber = 0
    var attempts = 0
    var maxAttempts = 10
    var minRange = 1
    var maxRange = 100
    
    function start()
        system.out.println("=== 猜数字游戏 ===")
        system.out.println("我想了一个 " + to_string(this.minRange) + " 到 " + to_string(this.maxRange) + " 之间的数字")
        system.out.println("你有 " + to_string(this.maxAttempts) + " 次机会猜中它！")
        system.out.println()
        
        # 生成随机数
        this.secretNumber = this.randomInt(this.minRange, this.maxRange)
        this.attempts = 0
        
        # 游戏循环
        loop
            if this.attempts >= this.maxAttempts
                system.out.println("游戏结束！你已用完所有机会。")
                system.out.println("正确答案是: " + to_string(this.secretNumber))
                break
            end
            
            # 获取用户输入
            system.out.print("第 " + to_string(this.attempts + 1) + " 次猜测: ")
            var input = system.in.getline()
            
            var guess = 0
            try
                guess = to_integer(input)
            catch e
                system.out.println("请输入有效的数字！")
                continue
            end
            
            # 检查范围
            if guess < this.minRange || guess > this.maxRange
                system.out.println("请输入 " + to_string(this.minRange) + " 到 " + to_string(this.maxRange) + " 之间的数字！")
                continue
            end
            
            this.attempts += 1
            
            # 检查猜测
            if guess == this.secretNumber
                system.out.println("恭喜你！猜对了！")
                system.out.println("你用了 " + to_string(this.attempts) + " 次就猜中了！")
                
                # 评分
                if this.attempts <= 3
                    system.out.println("评价：太厉害了！")
                else if this.attempts <= 6
                    system.out.println("评价：不错！")
                else
                    system.out.println("评价：还需要努力！")
                end
                
                break
            else if guess < this.secretNumber
                system.out.println("太小了！还有 " + to_string(this.maxAttempts - this.attempts) + " 次机会")
            else
                system.out.println("太大了！还有 " + to_string(this.maxAttempts - this.attempts) + " 次机会")
            end
            
            # 提示范围
            if this.attempts < this.maxAttempts
                var remaining = this.maxAttempts - this.attempts
                if remaining == 1
                    system.out.println("提示：这是你最后一次机会了！")
                else if remaining == 3
                    system.out.println("提示：还剩 3 次机会，仔细想想！")
                end
            end
        end
        
        # 询问是否再玩一次
        system.out.print("\n要再玩一次吗？(y/n): ")
        var again = system.in.getline()
        if again == "y" || again == "Y"
            system.out.println("\n")
            this.start()
        else
            system.out.println("感谢游戏！再见！")
        end
    end
    
    function randomInt(min, max)
        return math.randint(min, max)
    end
end

# 启动游戏
var game = new GuessNumberGame{}
game.start()
```

## 2.15.3 数据库应用

### 案例：简单的任务管理系统

使用数据库实现一个任务管理应用。

```covscript
import csdbc

class TaskManager
    var db = null
    
    function construct(dbPath) override
        this.db = csdbc.connect("sqlite", dbPath)
        this.initDatabase()
    end
    
    function initDatabase()
        # 创建任务表
        this.db.execute("
            CREATE TABLE IF NOT EXISTS tasks (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                title TEXT NOT NULL,
                description TEXT,
                status TEXT DEFAULT 'pending',
                priority INTEGER DEFAULT 0,
                created_at INTEGER,
                completed_at INTEGER
            )
        ")
        
        # 创建标签表
        this.db.execute("
            CREATE TABLE IF NOT EXISTS tags (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT UNIQUE NOT NULL
            )
        ")
        
        # 创建任务-标签关联表
        this.db.execute("
            CREATE TABLE IF NOT EXISTS task_tags (
                task_id INTEGER,
                tag_id INTEGER,
                FOREIGN KEY(task_id) REFERENCES tasks(id),
                FOREIGN KEY(tag_id) REFERENCES tags(id),
                PRIMARY KEY(task_id, tag_id)
            )
        ")
    end
    
    function addTask(title, description, priority)
        var timestamp = runtime.time()
        
        this.db.execute("
            INSERT INTO tasks (title, description, priority, created_at)
            VALUES (?, ?, ?, ?)
        ", {title, description, priority, timestamp})
        
        var taskId = this.db.lastInsertId()
        system.out.println("任务创建成功，ID: " + to_string(taskId))
        return taskId
    end
    
    function listTasks(status)
        var sql = "SELECT * FROM tasks"
        var params = new list
        
        if status != null
            sql += " WHERE status = ?"
            params.push_back(status)
        end
        
        sql += " ORDER BY priority DESC, created_at DESC"
        
        var tasks = this.db.query(sql, params)
        
        if tasks.size == 0
            system.out.println("没有找到任务")
            return
        end
        
        system.out.println("\n任务列表:")
        var separator = ""
        for i = 0, i < 60, ++i
            separator += "="
        end
        system.out.println(separator)
        
        foreach task in tasks
            this.printTask(task)
            var line = ""
            for i = 0, i < 60, ++i
                line += "-"
            end
            system.out.println(line)
        end
    end
    
    function printTask(task)
        system.out.println("ID: " + to_string(task["id"]))
        system.out.println("标题: " + task["title"])
        system.out.println("描述: " + task["description"])
        system.out.println("状态: " + task["status"])
        system.out.println("优先级: " + to_string(task["priority"]))
        
        # 获取标签
        var tags = this.getTaskTags(task["id"])
        if tags.size > 0
            var tagNames = new array
            foreach tag in tags
                tagNames.push_back(tag["name"])
            end
            system.out.println("标签: " + tagNames.join(", "))
        end
    end
    
    function updateTaskStatus(taskId, newStatus)
        var timestamp = null
        if newStatus == "completed"
            timestamp = runtime.time()
            this.db.execute("
                UPDATE tasks SET status = ?, completed_at = ?
                WHERE id = ?
            ", {newStatus, timestamp, taskId})
        else
            this.db.execute("
                UPDATE tasks SET status = ? WHERE id = ?
            ", {newStatus, taskId})
        end
        
        system.out.println("任务状态已更新")
    end
    
    function deleteTask(taskId)
        # 删除任务标签关联
        this.db.execute("DELETE FROM task_tags WHERE task_id = ?", {taskId})
        
        # 删除任务
        this.db.execute("DELETE FROM tasks WHERE id = ?", {taskId})
        
        system.out.println("任务已删除")
    end
    
    function addTag(taskId, tagName)
        # 获取或创建标签
        var tags = this.db.query("SELECT id FROM tags WHERE name = ?", {tagName})
        var tagId = null
        
        if tags.size > 0
            tagId = tags[0]["id"]
        else
            this.db.execute("INSERT INTO tags (name) VALUES (?)", {tagName})
            tagId = this.db.lastInsertId()
        end
        
        # 关联任务和标签
        try
            this.db.execute("INSERT INTO task_tags (task_id, tag_id) VALUES (?, ?)",
                          {taskId, tagId})
            system.out.println("标签已添加")
        catch e
            system.out.println("标签已存在")
        end
    end
    
    function getTaskTags(taskId)
        return this.db.query("
            SELECT t.* FROM tags t
            JOIN task_tags tt ON t.id = tt.tag_id
            WHERE tt.task_id = ?
        ", {taskId})
    end
    
    function searchTasks(keyword)
        var tasks = this.db.query("
            SELECT * FROM tasks
            WHERE title LIKE ? OR description LIKE ?
            ORDER BY priority DESC, created_at DESC
        ", {"%" + keyword + "%", "%" + keyword + "%"})
        
        system.out.println("\n搜索结果 (关键词: " + keyword + "):")
        system.out.println("找到 " + to_string(tasks.size) + " 个任务")
        var separator = ""
        for i = 0, i < 60, ++i
            separator += "="
        end
        system.out.println(separator)
        
        foreach task in tasks
            this.printTask(task)
            var line = ""
            for i = 0, i < 60, ++i
                line += "-"
            end
            system.out.println(line)
        end
    end
    
    function getStatistics()
        var total = this.db.query("SELECT COUNT(*) as count FROM tasks")[0]["count"]
        var pending = this.db.query("SELECT COUNT(*) as count FROM tasks WHERE status = 'pending'")[0]["count"]
        var completed = this.db.query("SELECT COUNT(*) as count FROM tasks WHERE status = 'completed'")[0]["count"]
        
        system.out.println("\n任务统计:")
        system.out.println("总任务数: " + to_string(total))
        system.out.println("待完成: " + to_string(pending))
        system.out.println("已完成: " + to_string(completed))
    end
    

    
    function close()
        if this.db != null
            this.db.close()
        end
    end
end

# 使用任务管理器
var taskMgr = new TaskManager{"tasks.db"}

# 添加任务
var task1 = taskMgr.addTask("完成报告", "需要在周五前完成月度报告", 5)
var task2 = taskMgr.addTask("购买食品", "去超市购买本周的食品", 3)
var task3 = taskMgr.addTask("学习 CovScript", "完成 CovScript 教程", 4)

# 添加标签
taskMgr.addTag(task1, "工作")
taskMgr.addTag(task2, "生活")
taskMgr.addTag(task3, "学习")

# 列出所有任务
taskMgr.listTasks(null)

# 更新任务状态
taskMgr.updateTaskStatus(task2, "completed")

# 列出待完成任务
taskMgr.listTasks("pending")

# 搜索任务
taskMgr.searchTasks("CovScript")

# 统计
taskMgr.getStatistics()

# 清理
taskMgr.close()
```

## 2.15.4 文件处理

### 案例：日志分析工具

分析日志文件并生成统计报告。

```covscript
class LogAnalyzer
    var logFile = ""
    var errorCount = 0
    var warningCount = 0
    var infoCount = 0
    var errors = new list
    var warnings = new list
    
    function construct(filename) override
        this.logFile = filename
    end
    
    function analyze()
        if !system.path.exist(this.logFile)
            system.out.println("文件不存在: " + this.logFile)
            return false
        end
        
        var file = iostream.fstream(this.logFile, iostream.openmode.in)
        var lineNumber = 0
        
        try
            loop
                var line = file.getline()
                if file.eof()
                    break
                end
                
                lineNumber += 1
                this.parseLine(line, lineNumber)
            end
        catch e
            system.out.println("分析错误: " + to_string(e))
            return false
        finally
            file.close()
        end
        
        return true
    end
    
    function parseLine(line, lineNum)
        # 检测日志级别
        if this.contains(line, "ERROR")
            this.errorCount += 1
            this.errors.push_back({
                "line": lineNum,
                "content": line
            })
        else if this.contains(line, "WARNING") || this.contains(line, "WARN")
            this.warningCount += 1
            this.warnings.push_back({
                "line": lineNum,
                "content": line
            })
        else if this.contains(line, "INFO")
            this.infoCount += 1
        end
    end
    
    function generateReport()
        system.out.println("\n===== 日志分析报告 =====")
        system.out.println("文件: " + this.logFile)
        system.out.println("\n统计:")
        system.out.println("  错误 (ERROR): " + to_string(this.errorCount))
        system.out.println("  警告 (WARNING): " + to_string(this.warningCount))
        system.out.println("  信息 (INFO): " + to_string(this.infoCount))
        
        if this.errors.size > 0
            system.out.println("\n错误详情:")
            var count = 0
            foreach error in this.errors
                count += 1
                if count > 10  # 只显示前 10 个错误
                    system.out.println("  ... 还有 " + to_string(this.errors.size - 10) + " 个错误")
                    break
                end
                system.out.println("  行 " + to_string(error["line"]) + ": " + error["content"])
            end
        end
        
        if this.warnings.size > 0
            system.out.println("\n警告详情:")
            var count = 0
            foreach warning in this.warnings
                count += 1
                if count > 5  # 只显示前 5 个警告
                    system.out.println("  ... 还有 " + to_string(this.warnings.size - 5) + " 个警告")
                    break
                end
                system.out.println("  行 " + to_string(warning["line"]) + ": " + warning["content"])
            end
        end
        
        system.out.println("\n========================")
    end
    
    function saveReport(outputFile)
        var out = iostream.fstream(outputFile, iostream.openmode.out)
        
        try
            out.println("日志分析报告")
            out.println("文件: " + this.logFile)
            out.println("生成时间: " + to_string(runtime.time()))
            out.println("")
            out.println("统计:")
            out.println("  错误: " + to_string(this.errorCount))
            out.println("  警告: " + to_string(this.warningCount))
            out.println("  信息: " + to_string(this.infoCount))
            out.println("")
            
            if this.errors.size > 0
                out.println("所有错误:")
                foreach error in this.errors
                    out.println("  行 " + to_string(error["line"]) + ": " + error["content"])
                end
            end
            
            system.out.println("报告已保存到: " + outputFile)
        catch e
            system.out.println("保存报告失败: " + to_string(e))
        finally
            out.close()
        end
    end
    
    function contains(str, substr)
        return str.find(substr, 0) != -1
    end
end

# 使用日志分析器
var analyzer = new LogAnalyzer{"application.log"}

if analyzer.analyze()
    analyzer.generateReport()
    analyzer.saveReport("report.txt")
else
    system.out.println("分析失败")
end
```

### 案例：文件批量重命名工具

```covscript
class FileRenamer
    var directory = ""
    var pattern = ""
    var replacement = ""
    var dryRun = true
    
    function construct(dir) override
        this.directory = dir
    end
    
    function setPattern(p, r)
        this.pattern = p
        this.replacement = r
    end
    
    function setDryRun(dry)
        this.dryRun = dry
    end
    
    function rename()
        if !system.path.exist(this.directory)
            system.out.println("目录不存在: " + this.directory)
            return
        end
        
        # 获取目录中的文件
        var files = system.path.scan(this.directory)
        var renameCount = 0
        
        system.out.println("扫描目录: " + this.directory)
        system.out.println("模式: " + this.pattern + " -> " + this.replacement)
        
        if this.dryRun
            system.out.println("(预览模式，不会实际重命名)")
        end
        
        system.out.println("")
        
        foreach file in files
            # 跳过目录
            var fullPath = this.directory + "/" + file
            if system.path.is_dir(fullPath)
                continue
            end
            
            # 检查是否匹配模式
            if this.contains(file, this.pattern)
                var newName = this.replace(file, this.pattern, this.replacement)
                var newPath = this.directory + "/" + newName
                
                system.out.println(file + " -> " + newName)
                
                if !this.dryRun
                    try
                        system.file.rename(fullPath, newPath)
                        renameCount += 1
                    catch e
                        system.out.println("  错误: " + to_string(e))
                    end
                else
                    renameCount += 1
                end
            end
        end
        
        system.out.println("")
        system.out.println("处理完成，" + to_string(renameCount) + " 个文件" +
                         (this.dryRun ? "将被" : "已") + "重命名")
    end
    
    function contains(str, substr)
        return str.find(substr, 0) != -1
    end
    
    function replace(str, pattern, replacement)
        var result = ""
        var i = 0
        
        loop
            if i >= str.size
                break
            end
            
            var found = true
            for j = 0, j < pattern.size && (i + j) < str.size, ++j
                if str[i + j] != pattern[j]
                    found = false
                    break
                end
            end
            
            if found
                result += replacement
                i += pattern.size
            else
                result += str[i]
                i += 1
            end
        end
        
        return result
    end
end

# 使用文件重命名工具
var renamer = new FileRenamer{"./documents"}

# 设置重命名规则
renamer.setPattern("old", "new")

# 先预览
renamer.setDryRun(true)
renamer.rename()

# 确认后执行
system.out.print("\n确认重命名？(y/n): ")
var confirm = system.in.getline()

if confirm == "y" || confirm == "Y"
    renamer.setDryRun(false)
    renamer.rename()
else
    system.out.println("操作已取消")
end
```

## 总结

本章展示了 CovScript 在各种实际场景中的应用：

- **网络应用**：HTTP 服务器、TCP 聊天室
- **游戏开发**：贪吃蛇、猜数字游戏
- **数据库应用**：任务管理系统
- **文件处理**：日志分析、批量重命名

这些案例演示了：
1. 如何组织较大规模的 CovScript 项目
2. 如何使用面向对象设计模式
3. 如何处理网络、文件、数据库等资源
4. 如何进行错误处理和资源清理
5. 如何构建用户友好的交互界面

通过学习这些案例，你可以更好地理解如何用 CovScript 开发实际应用程序。
