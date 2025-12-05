# 2.14 高级库与生态

CovScript 拥有丰富的扩展库生态系统，提供网络编程、数据库操作、GUI 开发等功能。本章介绍 CovScript 生态中的主要库及其使用方法。

## 2.14.1 covscript-network - 网络编程

`covscript-network` 提供了 TCP 和 UDP 网络通信功能。

### 安装

```bash
cspkg install network
```

### TCP 服务器

```covscript
import network

# 创建 TCP 服务器
var server = network.tcp.server()

# 绑定到地址和端口
server.bind("127.0.0.1", 8080)

# 开始监听
server.listen(5)  # 最大连接队列长度

system.out.println("服务器启动在 127.0.0.1:8080")

loop
    # 接受客户端连接
    var client = server.accept()
    system.out.println("客户端已连接")
    
    try
        # 接收数据
        var data = client.recv(1024)
        system.out.println("收到: " + data)
        
        # 发送响应
        var response = "Hello from server!"
        client.send(response)
        
    catch e
        system.out.println("错误: " + to_string(e))
    finally
        # 关闭连接
        client.close()
    end
end

server.close()
```

### TCP 客户端

```covscript
import network

# 创建 TCP 客户端
var client = network.tcp.client()

try
    # 连接到服务器
    client.connect("127.0.0.1", 8080)
    system.out.println("已连接到服务器")
    
    # 发送数据
    var message = "Hello, Server!"
    client.send(message)
    system.out.println("已发送: " + message)
    
    # 接收响应
    var response = client.recv(1024)
    system.out.println("收到响应: " + response)
    
catch e
    system.out.println("连接失败: " + to_string(e))
finally
    client.close()
end
```

### UDP 通信

```covscript
import network

# UDP 服务器
function udpServer()
    var sock = network.udp.socket()
    sock.bind("127.0.0.1", 9000)
    
    system.out.println("UDP 服务器启动在 127.0.0.1:9000")
    
    for i = 0, i < 5, ++i
        # 接收数据和客户端地址
        var result = sock.recvfrom(1024)
        var data = result[0]
        var addr = result[1]
        var port = result[2]
        
        system.out.println("从 " + addr + ":" + to_string(port) + " 收到: " + data)
        
        # 发送回复
        var response = "收到消息: " + data
        sock.sendto(response, addr, port)
    end
    
    sock.close()
end

# UDP 客户端
function udpClient()
    var sock = network.udp.socket()
    
    var serverAddr = "127.0.0.1"
    var serverPort = 9000
    
    for i = 1, i <= 5, ++i
        # 发送数据
        var message = "消息 " + to_string(i)
        sock.sendto(message, serverAddr, serverPort)
        system.out.println("已发送: " + message)
        
        # 接收响应
        var result = sock.recvfrom(1024)
        var response = result[0]
        system.out.println("收到响应: " + response)
        
        runtime.sleep(1000)
    end
    
    sock.close()
end
```

### 多客户端聊天服务器

```covscript
import network
import fiber

class ChatServer
    var server = null
    var clients = new list
    var running = false
    
    function start(host, port)
        this.server = network.tcp.server()
        this.server.bind(host, port)
        this.server.listen(10)
        this.running = true
        
        system.out.println("聊天服务器启动在 " + host + ":" + to_string(port))
        
        # 接受连接的主循环
        loop
            if !this.running
                break
            end
            
            try
                var client = this.server.accept()
                this.clients.push_back(client)
                system.out.println("新客户端连接，总数: " + to_string(this.clients.size))
                
                # 为每个客户端创建处理协程
                this.handleClient(client)
            catch e
                system.out.println("错误: " + to_string(e))
            end
        end
    end
    
    function handleClient(client)
        # 创建客户端处理协程
        var handler = fiber.create([](c, server) {
            loop
                try
                    var data = c.recv(1024)
                    if data.empty
                        break
                    end
                    
                    system.out.println("收到消息: " + data)
                    
                    # 广播给所有客户端
                    server.broadcast(data)
                catch e
                    system.out.println("客户端错误: " + to_string(e))
                    break
                end
                
                fiber.yield()
            end
            
            # 移除断开的客户端
            c.close()
        }, client, this)
        
        fiber.resume(handler)
    end
    
    function broadcast(message)
        foreach client in this.clients
            try
                client.send(message)
            catch e
                # 忽略发送失败的客户端
            end
        end
    end
    
    function stop()
        this.running = false
        if this.server != null
            this.server.close()
        end
    end
end
```

## 2.14.2 csdbc - 数据库操作

`csdbc` 提供了统一的数据库访问接口，支持 SQLite、MySQL、PostgreSQL 等数据库。

### 安装

```bash
cspkg install csdbc
```

### SQLite 基础操作

```covscript
import csdbc

# 连接到 SQLite 数据库
var db = csdbc.connect("sqlite", "mydata.db")

try
    # 创建表
    db.execute("CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        email TEXT UNIQUE,
        age INTEGER
    )")
    
    # 插入数据
    db.execute("INSERT INTO users (name, email, age) VALUES (?, ?, ?)",
               {"Alice", "alice@example.com", 25})
    db.execute("INSERT INTO users (name, email, age) VALUES (?, ?, ?)",
               {"Bob", "bob@example.com", 30})
    db.execute("INSERT INTO users (name, email, age) VALUES (?, ?, ?)",
               {"Charlie", "charlie@example.com", 35})
    
    system.out.println("数据插入成功")
    
    # 查询数据
    var result = db.query("SELECT * FROM users")
    
    system.out.println("用户列表:")
    foreach row in result
        system.out.println("ID: " + to_string(row["id"]) +
                         ", 姓名: " + row["name"] +
                         ", 邮箱: " + row["email"] +
                         ", 年龄: " + to_string(row["age"]))
    end
    
    # 更新数据
    db.execute("UPDATE users SET age = ? WHERE name = ?", {26, "Alice"})
    system.out.println("数据更新成功")
    
    # 删除数据
    db.execute("DELETE FROM users WHERE age > ?", {30})
    system.out.println("数据删除成功")
    
catch e
    system.out.println("数据库错误: " + to_string(e))
finally
    # 关闭连接
    db.close()
end
```

### 事务处理

```covscript
import csdbc

var db = csdbc.connect("sqlite", "mydata.db")

try
    # 开始事务
    db.beginTransaction()
    
    # 执行多个操作
    db.execute("INSERT INTO accounts (name, balance) VALUES (?, ?)", {"账户A", 1000})
    db.execute("INSERT INTO accounts (name, balance) VALUES (?, ?)", {"账户B", 500})
    
    # 转账操作
    db.execute("UPDATE accounts SET balance = balance - ? WHERE name = ?", {200, "账户A"})
    db.execute("UPDATE accounts SET balance = balance + ? WHERE name = ?", {200, "账户B"})
    
    # 提交事务
    db.commit()
    system.out.println("事务提交成功")
    
catch e
    # 发生错误时回滚
    db.rollback()
    system.out.println("事务回滚: " + to_string(e))
finally
    db.close()
end
```

### MySQL 连接

```covscript
import csdbc

# 连接到 MySQL
var db = csdbc.connect("mysql", {
    "host": "localhost",
    "port": 3306,
    "user": "root",
    "password": "password",
    "database": "mydb"
})

try
    # 执行查询
    var result = db.query("SELECT * FROM products WHERE price > ?", {100})
    
    foreach product in result
        system.out.println("产品: " + product["name"] +
                         ", 价格: " + to_string(product["price"]))
    end
    
catch e
    system.out.println("查询失败: " + to_string(e))
finally
    db.close()
end
```

### ORM 风格的数据访问

```covscript
import csdbc

class UserModel
    var db = null
    
    function construct(database)
        this.db = database
    end
    
    function findAll()
        return this.db.query("SELECT * FROM users")
    end
    
    function findById(id)
        var result = this.db.query("SELECT * FROM users WHERE id = ?", {id})
        if result.size > 0
            return result[0]
        else
            return null
        end
    end
    
    function findByName(name)
        return this.db.query("SELECT * FROM users WHERE name LIKE ?", {"%" + name + "%"})
    end
    
    function create(name, email, age)
        this.db.execute("INSERT INTO users (name, email, age) VALUES (?, ?, ?)",
                       {name, email, age})
        return this.db.lastInsertId()
    end
    
    function update(id, data)
        var sets = new list
        var values = new list
        
        foreach key in data.keys()
            sets.push_back(key + " = ?")
            values.push_back(data[key])
        end
        
        values.push_back(id)
        
        var sql = "UPDATE users SET " + join(sets, ", ") + " WHERE id = ?"
        this.db.execute(sql, values)
    end
    
    function delete(id)
        this.db.execute("DELETE FROM users WHERE id = ?", {id})
    end
end

# 使用模型
var db = csdbc.connect("sqlite", "mydata.db")
var userModel = new UserModel(db)

# 创建用户
var userId = userModel.create("David", "david@example.com", 28)
system.out.println("创建用户 ID: " + to_string(userId))

# 查询用户
var user = userModel.findById(userId)
if user != null
    system.out.println("找到用户: " + user["name"])
end

# 更新用户
userModel.update(userId, {"age": 29})

# 删除用户
userModel.delete(userId)

db.close()
```

## 2.14.3 covscript-imgui - GUI 编程

`covscript-imgui` 是基于 Dear ImGui 的图形用户界面库，适合创建工具和调试界面。

### 安装

```bash
cspkg install imgui
```

### 基本窗口

```covscript
import imgui

# 初始化窗口
imgui.init("我的应用", 800, 600)

# 主循环
loop
    # 开始新帧
    if !imgui.beginFrame()
        break  # 窗口关闭
    end
    
    # 创建窗口
    imgui.begin("Hello Window")
    
    # 显示文本
    imgui.text("Hello, CovScript!")
    imgui.text("这是一个 ImGui 窗口")
    
    # 按钮
    if imgui.button("点击我")
        system.out.println("按钮被点击!")
    end
    
    imgui.end()
    
    # 渲染帧
    imgui.endFrame()
end

# 清理
imgui.shutdown()
```

### 交互控件

```covscript
import imgui

imgui.init("控件示例", 800, 600)

# 状态变量
var counter = 0
var text = "输入文本"
var slider_value = 0.5
var checkbox_state = false
var color = {1.0, 0.0, 0.0, 1.0}  # RGBA

loop
    if !imgui.beginFrame()
        break
    end
    
    imgui.begin("控件演示")
    
    # 文本输入
    text = imgui.inputText("文本框", text)
    
    # 滑动条
    slider_value = imgui.slider("滑动条", slider_value, 0.0, 1.0)
    imgui.text("值: " + to_string(slider_value))
    
    # 复选框
    checkbox_state = imgui.checkbox("复选框", checkbox_state)
    
    # 计数器
    imgui.text("计数: " + to_string(counter))
    if imgui.button("增加")
        counter += 1
    end
    imgui.sameLine()
    if imgui.button("减少")
        counter -= 1
    end
    
    # 颜色选择器
    color = imgui.colorEdit("颜色", color)
    
    # 分隔线
    imgui.separator()
    
    # 显示 FPS
    imgui.text("FPS: " + to_string(imgui.getFPS()))
    
    imgui.end()
    
    imgui.endFrame()
end

imgui.shutdown()
```

### 复杂布局

```covscript
import imgui

imgui.init("布局示例", 1200, 800)

var leftPanelWidth = 300
var items = {"项目 1", "项目 2", "项目 3", "项目 4", "项目 5"}
var selectedItem = 0

loop
    if !imgui.beginFrame()
        break
    end
    
    # 主菜单栏
    if imgui.beginMenuBar()
        if imgui.beginMenu("文件")
            if imgui.menuItem("打开")
                system.out.println("打开文件")
            end
            if imgui.menuItem("保存")
                system.out.println("保存文件")
            end
            imgui.separator()
            if imgui.menuItem("退出")
                break
            end
            imgui.endMenu()
        end
        
        if imgui.beginMenu("编辑")
            if imgui.menuItem("撤销")
                system.out.println("撤销")
            end
            if imgui.menuItem("重做")
                system.out.println("重做")
            end
            imgui.endMenu()
        end
        
        imgui.endMenuBar()
    end
    
    # 左侧面板
    imgui.begin("侧边栏", leftPanelWidth, -1)
    
    imgui.text("导航")
    imgui.separator()
    
    # 列表框
    selectedItem = imgui.listBox("项目列表", selectedItem, items)
    
    if imgui.button("添加项目")
        var newItem = "项目 " + to_string(items.size + 1)
        items.push_back(newItem)
    end
    
    imgui.end()
    
    # 主内容区域
    imgui.setNextWindowPos(leftPanelWidth, 0)
    imgui.setNextWindowSize(1200 - leftPanelWidth, 800)
    imgui.begin("主区域")
    
    imgui.text("选中的项目: " + items[selectedItem])
    
    # 子窗口
    imgui.beginChild("内容区", 0, 400, true)
    
    for i = 0, i < 50, ++i
        imgui.text("内容行 " + to_string(i))
    end
    
    imgui.endChild()
    
    imgui.end()
    
    imgui.endFrame()
end

imgui.shutdown()
```

## 2.14.4 ecs - 实体组件系统

ECS（Entity Component System）是一种用于游戏开发和模拟的架构模式。

### 基本概念

```covscript
import ecs

# 创建 ECS 世界
var world = ecs.createWorld()

# 定义组件
class Position
    var x = 0.0
    var y = 0.0
    
    function construct(px, py)
        this.x = px
        this.y = py
    end
end

class Velocity
    var dx = 0.0
    var dy = 0.0
    
    function construct(vx, vy)
        this.dx = vx
        this.dy = vy
    end
end

class Renderable
    var sprite = ""
    
    function construct(s)
        this.sprite = s
    end
end

# 创建实体
var entity1 = world.createEntity()
world.addComponent(entity1, new Position(100, 100))
world.addComponent(entity1, new Velocity(5, 0))
world.addComponent(entity1, new Renderable("player.png"))

var entity2 = world.createEntity()
world.addComponent(entity2, new Position(200, 200))
world.addComponent(entity2, new Velocity(-3, 2))
world.addComponent(entity2, new Renderable("enemy.png"))

# 定义系统
class MovementSystem
    function update(world, deltaTime)
        var entities = world.getEntitiesWith(Position, Velocity)
        
        foreach entity in entities
            var pos = world.getComponent(entity, Position)
            var vel = world.getComponent(entity, Velocity)
            
            pos.x += vel.dx * deltaTime
            pos.y += vel.dy * deltaTime
        end
    end
end

class RenderSystem
    function update(world, deltaTime)
        var entities = world.getEntitiesWith(Position, Renderable)
        
        foreach entity in entities
            var pos = world.getComponent(entity, Position)
            var ren = world.getComponent(entity, Renderable)
            
            # 渲染逻辑
            system.out.println("渲染 " + ren.sprite + " 于 (" +
                             to_string(pos.x) + ", " + to_string(pos.y) + ")")
        end
    end
end

# 注册系统
var movementSystem = new MovementSystem()
var renderSystem = new RenderSystem()

# 游戏循环
for frame = 0, frame < 10, ++frame
    var deltaTime = 0.016  # 约 60 FPS
    
    # 更新系统
    movementSystem.update(world, deltaTime)
    renderSystem.update(world, deltaTime)
    
    system.out.println("--- 帧 " + to_string(frame) + " ---")
    runtime.sleep(16)
end
```

### 游戏示例：简单的射击游戏

```covscript
import ecs

# 组件定义
class Transform
    var x = 0.0
    var y = 0.0
    var rotation = 0.0
end

class Health
    var current = 100
    var max = 100
end

class Weapon
    var damage = 10
    var fireRate = 0.5
    var lastFire = 0.0
end

class Enemy
    var speed = 50.0
end

# 创建游戏世界
var gameWorld = ecs.createWorld()

# 创建玩家
function createPlayer(world, x, y)
    var player = world.createEntity()
    world.addComponent(player, new Transform())
    world.getComponent(player, Transform).x = x
    world.getComponent(player, Transform).y = y
    world.addComponent(player, new Health())
    world.addComponent(player, new Weapon())
    return player
end

# 创建敌人
function createEnemy(world, x, y)
    var enemy = world.createEntity()
    world.addComponent(enemy, new Transform())
    world.getComponent(enemy, Transform).x = x
    world.getComponent(enemy, Transform).y = y
    world.addComponent(enemy, new Health())
    world.getComponent(enemy, Health).current = 50
    world.getComponent(enemy, Health).max = 50
    world.addComponent(enemy, new Enemy())
    return enemy
end

# 系统：敌人移动
class EnemyMovementSystem
    function update(world, deltaTime)
        var enemies = world.getEntitiesWith(Transform, Enemy)
        
        foreach enemy in enemies
            var transform = world.getComponent(enemy, Transform)
            var enemyComp = world.getComponent(enemy, Enemy)
            
            # 向玩家移动（简化版，向左移动）
            transform.x -= enemyComp.speed * deltaTime
        end
    end
end

# 系统：碰撞检测
class CollisionSystem
    function update(world, deltaTime)
        # 简化的碰撞检测
        system.out.println("检测碰撞...")
    end
end

# 初始化游戏
var player = createPlayer(gameWorld, 100, 300)
for i = 0, i < 5, ++i
    createEnemy(gameWorld, 800 + i * 100, 300 + (i * 50))
end

# 游戏循环
var enemySystem = new EnemyMovementSystem()
var collisionSystem = new CollisionSystem()

for tick = 0, tick < 100, ++tick
    var dt = 0.016
    
    enemySystem.update(gameWorld, dt)
    collisionSystem.update(gameWorld, dt)
    
    runtime.sleep(16)
end
```

## 2.14.5 cspkg - 包管理工具

`cspkg` 是 CovScript 的包管理器，用于安装、更新和管理扩展库。

### 基本命令

```bash
# 安装包
cspkg install <package_name>

# 安装特定版本
cspkg install <package_name>@<version>

# 更新包
cspkg update <package_name>

# 更新所有包
cspkg update --all

# 卸载包
cspkg uninstall <package_name>

# 搜索包
cspkg search <keyword>

# 列出已安装的包
cspkg list

# 查看包信息
cspkg info <package_name>
```

### 创建自己的包

创建 `package.json` 文件：

```json
{
    "name": "my-package",
    "version": "1.0.0",
    "description": "我的 CovScript 包",
    "author": "Your Name",
    "license": "MIT",
    "main": "main.csc",
    "dependencies": {
        "network": "^1.0.0"
    },
    "keywords": ["network", "tool"],
    "repository": {
        "type": "git",
        "url": "https://github.com/username/my-package"
    }
}
```

包的目录结构：

```
my-package/
├── package.json
├── main.csc          # 主入口文件
├── README.md
├── LICENSE
└── src/
    ├── module1.csc
    └── module2.csc
```

### 使用本地包

```covscript
# 在项目中导入本地包
import my_package

# 使用包中的功能
my_package.someFunction()
```

### 发布包

```bash
# 登录 cspkg 账户
cspkg login

# 发布包
cspkg publish

# 发布特定版本
cspkg publish --tag beta
```

## 2.14.6 其他实用库

### regex - 正则表达式

```covscript
import regex

# 创建正则表达式
var pattern = regex.compile("[0-9]+")

# 匹配
var text = "我有 123 个苹果和 456 个橙子"
var matches = pattern.findAll(text)

foreach match in matches
    system.out.println("找到数字: " + match)
end

# 替换
var result = pattern.replace(text, "***")
system.out.println(result)  # "我有 *** 个苹果和 *** 个橙子"
```

### json - JSON 处理

```covscript
import json

# 解析 JSON
var jsonStr = '{"name": "Alice", "age": 25, "skills": ["C++", "Python"]}'
var data = json.parse(jsonStr)

system.out.println("姓名: " + data["name"])
system.out.println("年龄: " + to_string(data["age"]))

# 生成 JSON
var obj = new hash_map
obj.insert("name", "Bob")
obj.insert("age", 30)
obj.insert("active", true)

var jsonOutput = json.stringify(obj)
system.out.println(jsonOutput)

# 格式化输出
var prettyJson = json.stringify(obj, 2)  # 缩进 2 空格
system.out.println(prettyJson)
```

### crypto - 加密解密

```covscript
import crypto

# MD5 哈希
var text = "Hello, World!"
var md5Hash = crypto.md5(text)
system.out.println("MD5: " + md5Hash)

# SHA256 哈希
var sha256Hash = crypto.sha256(text)
system.out.println("SHA256: " + sha256Hash)

# Base64 编码/解码
var encoded = crypto.base64Encode(text)
system.out.println("Base64: " + encoded)

var decoded = crypto.base64Decode(encoded)
system.out.println("解码: " + decoded)
```

### datetime - 日期时间

```covscript
import datetime

# 获取当前时间
var now = datetime.now()
system.out.println("当前时间: " + now.format("%Y-%m-%d %H:%M:%S"))

# 解析日期
var date = datetime.parse("2025-12-05", "%Y-%m-%d")
system.out.println("解析的日期: " + to_string(date.year) + "-" +
                 to_string(date.month) + "-" + to_string(date.day))

# 日期计算
var tomorrow = now.addDays(1)
var nextWeek = now.addDays(7)

# 日期比较
if tomorrow > now
    system.out.println("明天在今天之后")
end

# 时间戳转换
var timestamp = now.toTimestamp()
var fromTimestamp = datetime.fromTimestamp(timestamp)
```

## 2.14.7 库开发最佳实践

### 1. 模块化设计

```covscript
# mylib.csc
namespace mylib
    # 私有函数
    function _internalHelper(x)
        return x * 2
    end
    
    # 公共 API
    function publicFunction(x)
        return _internalHelper(x) + 1
    end
    
    # 常量
    constant VERSION = "1.0.0"
    constant MAX_SIZE = 1000
end

# 导出命名空间
export mylib
```

### 2. 文档化

```covscript
# 为函数添加文档注释
# @brief 计算两个数的和
# @param a 第一个数字
# @param b 第二个数字
# @return 两数之和
function add(a, b)
    return a + b
end
```

### 3. 错误处理

```covscript
# 良好的错误处理
function safeOperation(param)
    if param == null
        throw "参数不能为 null"
    end
    
    if typeid param != typeid 0
        throw "参数必须是数字类型"
    end
    
    try
        # 执行操作
        return param * 2
    catch e
        system.out.println("操作失败: " + to_string(e))
        return null
    end
end
```

### 4. 版本兼容性

```covscript
# 检查 CovScript 版本
if runtime.version() < "4.0.0"
    throw "此库需要 CovScript 4.0.0 或更高版本"
end

# 特性检测
function hasFeature(feature)
    try
        # 尝试使用特性
        return true
    catch e
        return false
    end
end
```

## 总结

CovScript 的生态系统提供了丰富的扩展库：

- **covscript-network**：完整的网络编程支持
- **csdbc**：统一的数据库访问接口
- **covscript-imgui**：即时模式 GUI 开发
- **ecs**：实体组件系统架构
- **cspkg**：便捷的包管理工具

以及其他实用库如 regex、json、crypto、datetime 等。通过这些库，可以快速开发各种类型的应用程序。
