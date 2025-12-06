# 2.14 高级库与生态

CovScript 拥有丰富的扩展库生态系统，提供网络编程、数据库操作、GUI 开发等功能。本章介绍 CovScript 生态中的主要库及其使用方法。

## 2.14.1 covscript-network - 网络编程

`covscript-network` 提供了 TCP 和 UDP 网络通信功能。

### 安装

```bash
cspkg install network
```

### 异步网络编程

CovScript Network 支持异步 I/O 操作，可以高效处理多个并发连接。

```covscript
import network.tcp as tcp
import network.async as async

# 等待异步状态完成或超时
function wait_for(state, timeout)
    var start_time = runtime.time()
    loop
        # 轮询待处理的异步事件
        async.poll_once()
        # 检查是否超时
        if runtime.time() - start_time >= timeout
            break
        end
        # 减少 CPU 占用
        runtime.delay(10)
    until state.has_done()
    return state.has_done()
end

# 创建工作守卫以确保异步会话有效
var guard = new async.work_guard
var state = null

# 创建 TCP socket
var sock = new tcp.socket
# 创建 TCP acceptor 监听端口 1024
var acpt = tcp.acceptor(tcp.endpoint_v4(1024))

# 提交异步 accept 会话
state = async.accept(sock, acpt)
loop
    system.out.println("等待客户端连接...")
until wait_for(state, 1000)

# 处理错误
if state.get_error() != null
    system.out.println("错误: " + state.get_error())
    system.exit(-1)
end

system.out.println("连接已建立")

# 准备新的异步状态对象用于读取数据
state = new async.state

# 循环读取数据
loop
    # 提交异步读取操作，直到遇到分隔符 "\r\n"
    async.read_until(sock, state, "\r\n")
    loop
        system.out.println("接收数据...")
    until wait_for(state, 1000)
    
    if state.get_error() != null
        system.out.println("错误: " + state.get_error())
        system.exit(-1)
    end
    
    # 输出接收到的数据
    system.out.print(state.get_result())
end
```

### TCP 服务器（同步模式）

```covscript
import network
using network

# 创建 UDP socket
var sock = new udp.socket
sock.open_v4()
sock.bind(udp.endpoint_v4(8080))

system.out.println("TCP 服务器启动在端口 8080")

loop
    var ep = udp.endpoint_v4(0)
    var data = sock.receive_from(1024, ep)
    system.out.println("收到: " + data)
    
    # 发送响应
    sock.send_to("服务器收到: " + data, ep)
end
```

### TCP 客户端

```covscript
import network
using network

# 创建 TCP 客户端
var sock = new tcp.socket
var ep = tcp.endpoint_v4("127.0.0.1", 8080)

try
    sock.connect(ep)
    system.out.println("已连接到服务器")
    
    # 发送数据
    var message = "Hello, Server!\r\n"
    sock.send(message)
    system.out.println("已发送: " + message)
    
    # 接收响应
    var response = sock.receive(1024)
    system.out.println("收到响应: " + response)
    
catch e
    system.out.println("连接失败: " + to_string(e))
finally
    sock.close()
end
```

### UDP 通信

```covscript
import network
using network

# UDP 服务器
function udpServer()
    var sock = new udp.socket
    sock.open_v4()
    sock.bind(udp.endpoint_v4(9000))
    
    system.out.println("UDP 服务器启动在端口 9000")
    
    loop
        var ep = udp.endpoint_v4(0)
        var data = sock.receive_from(1024, ep)
        
        system.out.println("收到: " + data)
        
        # 发送回复
        sock.send_to("服务器收到: " + data, ep)
    end
end

# UDP 客户端
function udpClient()
    var sock = new udp.socket
    sock.open_v4()
    
    var serverEp = udp.endpoint_v4("127.0.0.1", 9000)
    
    for i = 1, i <= 5, ++i
        # 发送数据
        var message = "消息 " + to_string(i)
        sock.send_to(message, serverEp)
        system.out.println("已发送: " + message)
        
        # 接收响应
        var ep = udp.endpoint_v4(0)
        var response = sock.receive_from(1024, ep)
        system.out.println("收到响应: " + response)
        
        runtime.delay(1000)
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
                    if data.empty()
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
var userModel = new UserModel{db}

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
world.addComponent(entity1, new Position{100, 100})
world.addComponent(entity1, new Velocity{5, 0})
world.addComponent(entity1, new Renderable{"player.png"})

var entity2 = world.createEntity()
world.addComponent(entity2, new Position{200, 200})
world.addComponent(entity2, new Velocity{-3, 2})
world.addComponent(entity2, new Renderable{"enemy.png"})

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
var movementSystem = new MovementSystem{}
var renderSystem = new RenderSystem{}

# 游戏循环
for frame = 0, frame < 10, ++frame
    var deltaTime = 0.016  # 约 60 FPS
    
    # 更新系统
    movementSystem.update(world, deltaTime)
    renderSystem.update(world, deltaTime)
    
    system.out.println("--- 帧 " + to_string(frame) + " ---")
    runtime.delay(16)
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
    world.addComponent(player, new Transform{})
    world.getComponent(player, Transform).x = x
    world.getComponent(player, Transform).y = y
    world.addComponent(player, new Health{})
    world.addComponent(player, new Weapon{})
    return player
end

# 创建敌人
function createEnemy(world, x, y)
    var enemy = world.createEntity()
    world.addComponent(enemy, new Transform{})
    world.getComponent(enemy, Transform).x = x
    world.getComponent(enemy, Transform).y = y
    world.addComponent(enemy, new Health{})
    world.getComponent(enemy, Health).current = 50
    world.getComponent(enemy, Health).max = 50
    world.addComponent(enemy, new Enemy{})
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
var enemySystem = new EnemyMovementSystem{}
var collisionSystem = new CollisionSystem{}

for tick = 0, tick < 100, ++tick
    var dt = 0.016
    
    enemySystem.update(gameWorld, dt)
    collisionSystem.update(gameWorld, dt)
    
    runtime.delay(16)
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

## 2.14.6 covscript-gmssl - 国密算法支持

`covscript-gmssl` 提供了国密（GM/T）加密算法的支持，包括 SM2、SM3、SM4 等算法。

### 安装

```bash
cspkg install gmssl
```

### SM3 哈希算法

SM3 是中国国家密码管理局发布的密码杂凑算法。

```covscript
import gmssl

# 计算 SM3 哈希（返回字节数组）
var text = "Hello, CovScript!"
var hash_bytes = gmssl.sm3(gmssl.bytes_encode(text))
var hash_hex = gmssl.hex_encode(hash_bytes)
system.out.println("SM3 哈希: " + hash_hex)

# SM3 HMAC
var key = "secret_key"
var hmac = gmssl.sm3_hmac(gmssl.bytes_encode(key), gmssl.bytes_encode(text))
system.out.println("SM3 HMAC: " + gmssl.hex_encode(hmac))

# SM3 PBKDF2 - 基于密码的密钥派生
var password = "my_password"
var salt = "covscript"
var iterations = 5
var key_bytes = gmssl.sm3_pbkdf2(password, salt, iterations, gmssl.sm4_key_size)
system.out.println("派生密钥: " + gmssl.hex_encode(key_bytes))
```

### SM4 对称加密

SM4 是一种分组密码算法，分组长度为 128 位，密钥长度为 128 位。

```covscript
import gmssl

# 生成密钥和初始化向量
var key = gmssl.rand_bytes(gmssl.sm4_key_size)
var iv = gmssl.rand_bytes(gmssl.sm4_key_size)

# 使用 CTR 模式加密
var plaintext = "机密信息"
var plaintext_encoded = gmssl.base64_encode(plaintext)
var ciphertext = gmssl.sm4(gmssl.sm4_mode.ctr_encrypt, key, iv, gmssl.bytes_encode(plaintext_encoded))

system.out.println("加密后: " + gmssl.hex_encode(ciphertext))

# 解密数据
var decrypted_bytes = gmssl.sm4(gmssl.sm4_mode.ctr_decrypt, key, iv, ciphertext)
var decrypted = gmssl.base64_decode(gmssl.bytes_decode(decrypted_bytes))
system.out.println("解密后: " + decrypted)

# 使用 CBC 模式加密
var ciphertext_cbc = gmssl.sm4(gmssl.sm4_mode.cbc_encrypt, key, iv, gmssl.bytes_encode(plaintext_encoded))
var decrypted_cbc = gmssl.base64_decode(gmssl.bytes_decode(gmssl.sm4(gmssl.sm4_mode.cbc_decrypt, key, iv, ciphertext_cbc)))
system.out.println("CBC 解密: " + decrypted_cbc)
```

### SM2 非对称加密

SM2 是基于椭圆曲线的公钥密码算法，包括数字签名、密钥交换和公钥加密。

```covscript
import gmssl

# 生成 SM2 密钥对
var keypair = gmssl.sm2_keygen()
var publicKey = keypair.public_key
var privateKey = keypair.private_key

# 将密钥保存为 PEM 格式
gmssl.sm2_pem_write("public.pem", gmssl.pem_name_pbk, publicKey)
gmssl.sm2_pem_write("private.pem", gmssl.pem_name_pvk, privateKey)

# 从 PEM 文件读取密钥
var loaded_pubkey = gmssl.sm2_pem_read("public.pem", gmssl.pem_name_pbk)
var loaded_privkey = gmssl.sm2_pem_read("private.pem", gmssl.pem_name_pvk)

# 使用公钥加密
var message = "重要消息"
var encrypted = gmssl.sm2_encrypt(loaded_pubkey, gmssl.bytes_encode(message))
system.out.println("加密结果: " + gmssl.base64_encode(encrypted))

# 使用私钥解密（需要密码保护）
var keypass = "password"
var decrypted_bytes = gmssl.sm2_decrypt(loaded_privkey, keypass, encrypted)
var decrypted = gmssl.bytes_decode(decrypted_bytes)
system.out.println("解密结果: " + decrypted)

# 数字签名
var data = gmssl.bytes_encode("需要签名的数据")
var signature = gmssl.sm2_sign(loaded_privkey, keypass, data)
system.out.println("签名: " + gmssl.base64_encode(signature))

# 验证签名
var isValid = gmssl.sm2_verify(loaded_pubkey, data, signature)
if isValid
    system.out.println("签名验证成功")
else
    system.out.println("签名验证失败")
end
```

### 实际应用：基于国密的 TLS 实现

基于 `simple_tls.csp` 包的安全通信实现：

```covscript
import simple_tls
import network

# 服务器端
class TLSServer
    var server = null
    
    function start(pkey_path, vkey_path, keypass, addr, port)
        this.server = new simple_tls.server
        this.server.listen(pkey_path, vkey_path, keypass, addr, port)
        
        system.out.println("TLS 服务器启动")
        system.out.println("指纹: " + this.server.fingerprint())
        
        if this.server.accept()
            system.out.println("客户端已连接")
            return true
        else
            system.out.println("连接失败")
            return false
        end
    end
    
    function send(data)
        this.server.send(data)
    end
    
    function receive()
        return this.server.receive()
    end
    
    function close()
        this.server.close()
    end
end

# 客户端
class TLSClient
    var client = null
    
    function connect(addr, port, authorized_fingerprint)
        this.client = new simple_tls.client
        # 添加授权的服务器指纹
        this.client.authorized_keys.insert(authorized_fingerprint)
        
        if this.client.connect(addr, port)
            system.out.println("已连接到服务器")
            return true
        else
            system.out.println("连接失败")
            return false
        end
    end
    
    function send(data)
        this.client.send(data)
    end
    
    function receive()
        return this.client.receive()
    end
    
    function close()
        this.client.close()
    end
end

# 使用示例（服务器）
# 首先生成密钥对：
# var keypair = gmssl.sm2_keygen()
# gmssl.sm2_pem_write("server_pub.pem", gmssl.pem_name_pbk, keypair.public_key)
# gmssl.sm2_pem_write("server_priv.pem", gmssl.pem_name_pvk, keypair.private_key)

var server = new TLSServer{}
if server.start("server_pub.pem", "server_priv.pem", "password", "0.0.0.0", 8443)
    # 接收并回显消息
    var message = server.receive()
    system.out.println("收到: " + message)
    server.send("服务器收到: " + message)
    server.close()
end

# 使用示例（客户端）
var client = new TLSClient{}
var server_fingerprint = "..." # 从服务器获取的指纹
if client.connect("127.0.0.1", 8443, server_fingerprint)
    client.send("Hello, TLS Server!")
    var response = client.receive()
    system.out.println("响应: " + response)
    client.close()
end
```

### 编码工具函数

GMSSL 提供了多种编码/解码工具：

```covscript
import gmssl

# Base64 编码/解码
var text = "Hello, World!"
var encoded = gmssl.base64_encode(text)
system.out.println("Base64: " + encoded)
var decoded = gmssl.base64_decode(encoded)
system.out.println("解码: " + decoded)

# Hex 编码/解码
var data = gmssl.bytes_encode("test data")
var hex = gmssl.hex_encode(data)
system.out.println("Hex: " + hex)
var original = gmssl.hex_decode(hex)
system.out.println("原始: " + gmssl.bytes_decode(original))

# 字节与字符串转换
var str = "测试文本"
var bytes = gmssl.bytes_encode(str)
var str_back = gmssl.bytes_decode(bytes)
system.out.println("转换回: " + str_back)

# 生成随机字节
var random_bytes = gmssl.rand_bytes(16)
system.out.println("随机字节: " + gmssl.hex_encode(random_bytes))

# 使用种子生成字符
var seed = 2333
var random_chars = gmssl.rand_chars(16, seed)
system.out.println("随机字符: " + random_chars)
```

## 2.14.7 picasso-ui - 现代化图形界面库

Picasso UI 是 CovScript 的现代化图形用户界面框架，提供了声明式 UI 构建方式。

### 安装

```bash
cspkg install picasso
```

### 基本窗口

```covscript
import picasso

# 创建应用
var app = picasso.Application()

# 创建主窗口
var window = picasso.Window()
window.setTitle("Picasso UI 示例")
window.setSize(800, 600)

# 添加组件
var layout = picasso.VBoxLayout()

var label = picasso.Label("欢迎使用 Picasso UI")
label.setFontSize(24)
layout.add(label)

var button = picasso.Button("点击我")
button.onClick([]() {
    system.out.println("按钮被点击！")
})
layout.add(button)

window.setLayout(layout)
window.show()

# 运行应用
app.exec()
```

### 声明式 UI 构建

```covscript
import picasso

class TodoApp
    var items = new list
    var window = null
    var listView = null
    
    function construct()
        this.window = picasso.Window()
        this.window.setTitle("待办事项")
        this.window.setSize(400, 600)
        
        this.buildUI()
    end
    
    function buildUI()
        var layout = picasso.VBoxLayout()
        
        # 标题
        var title = picasso.Label("我的待办事项")
        title.setFontSize(20)
        title.setAlignment(picasso.Align.Center)
        layout.add(title)
        
        # 输入框和添加按钮
        var inputLayout = picasso.HBoxLayout()
        var input = picasso.TextEdit()
        input.setPlaceholder("输入新任务...")
        inputLayout.add(input)
        
        var addBtn = picasso.Button("添加")
        addBtn.onClick([](app, inp) {
            var text = inp.getText()
            if text.size > 0
                app.addItem(text)
                inp.clear()
            end
        }, this, input)
        inputLayout.add(addBtn)
        
        layout.add(inputLayout)
        
        # 任务列表
        this.listView = picasso.ListView()
        layout.add(this.listView)
        
        # 统计信息
        var statsLabel = picasso.Label("总计: 0 项")
        layout.add(statsLabel)
        
        this.window.setLayout(layout)
    end
    
    function addItem(text)
        this.items.push_back({
            "text": text,
            "completed": false
        })
        this.updateList()
    end
    
    function updateList()
        this.listView.clear()
        
        foreach item in this.items
            var itemWidget = picasso.HBoxLayout()
            
            var checkbox = picasso.CheckBox(item["text"])
            checkbox.setChecked(item["completed"])
            itemWidget.add(checkbox)
            
            var deleteBtn = picasso.Button("删除")
            itemWidget.add(deleteBtn)
            
            this.listView.addItem(itemWidget)
        end
    end
    
    function show()
        this.window.show()
    end
end

# 运行应用
var app = picasso.Application()
var todoApp = new TodoApp{}
todoApp.show()
app.exec()
```

### 响应式布局

```covscript
import picasso

# 创建响应式布局
var window = picasso.Window()
window.setTitle("响应式布局")

var layout = picasso.GridLayout(3, 3)  # 3x3 网格

# 添加按钮到网格
for i = 0, i < 9, ++i
    var btn = picasso.Button("按钮 " + to_string(i + 1))
    btn.onClick([](index) {
        system.out.println("点击了按钮 " + to_string(index))
    }, i + 1)
    layout.add(btn, i / 3, i % 3)
end

window.setLayout(layout)
window.show()

var app = picasso.Application()
app.exec()
```

### 自定义样式

```covscript
import picasso

# 设置应用主题
picasso.setTheme("dark")  # 或 "light"

var window = picasso.Window()
window.setTitle("自定义样式")

var button = picasso.Button("样式化按钮")

# 设置自定义样式
button.setStyle({
    "background-color": "#4CAF50",
    "color": "white",
    "border-radius": "5px",
    "padding": "10px 20px",
    "font-size": "16px"
})

# 悬停效果
button.onHover([]() {
    button.setStyle({"background-color": "#45a049"})
}, []() {
    button.setStyle({"background-color": "#4CAF50"})
})

window.add(button)
window.show()

var app = picasso.Application()
app.exec()
```

## 2.14.8 covanalysis - 静态代码分析工具

covanalysis 是 CovScript 的静态代码分析工具，用于检查代码质量、发现潜在问题和代码异味。

### 安装

```bash
cspkg install covanalysis
```

### 基本使用

```bash
# 分析单个文件
covanalysis analyze mycode.csc

# 分析整个项目
covanalysis analyze --recursive ./src

# 生成详细报告
covanalysis analyze mycode.csc --report=detailed

# 输出 JSON 格式
covanalysis analyze mycode.csc --format=json --output=report.json
```

### 支持的检查项

covanalysis 支持多种代码检查：

1. **语法检查**：检测语法错误和不规范的写法
2. **未使用变量检测**：找出定义但从未使用的变量
3. **类型安全**：检查潜在的类型错误
4. **代码复杂度**：计算函数的圈复杂度
5. **最佳实践**：检查是否遵循 CovScript 最佳实践
6. **性能提示**：识别可能的性能问题

### 在代码中使用

```covscript
import covanalysis

# 程序化分析代码
var analyzer = covanalysis.Analyzer()

# 设置检查规则
analyzer.enableRule("unused-variables")
analyzer.enableRule("complexity")
analyzer.setComplexityThreshold(10)

# 分析文件
var results = analyzer.analyzeFile("mycode.csc")

# 处理结果
foreach issue in results
    system.out.println("问题: " + issue["message"])
    system.out.println("位置: 第 " + to_string(issue["line"]) + " 行")
    system.out.println("严重程度: " + issue["severity"])
    system.out.println("")
end

# 生成报告
analyzer.generateReport("report.html", "html")
```

### 自定义检查规则

```covscript
import covanalysis

class CustomRule
    function check(node)
        # 检查逻辑
        if node.type == "function" && node.name.starts_with("_")
            return {
                "severity": "warning",
                "message": "私有函数应使用双下划线前缀",
                "line": node.line
            }
        end
        return null
    end
end

# 注册自定义规则
var analyzer = covanalysis.Analyzer()
analyzer.addCustomRule(new CustomRule{})
analyzer.analyzeFile("mycode.csc")
```

### 配置文件

在项目根目录创建 `.covanalysis.json` 配置文件：

```json
{
    "rules": {
        "unused-variables": "error",
        "complexity": "warning",
        "naming-convention": "warning",
        "line-length": "warning"
    },
    "thresholds": {
        "complexity": 15,
        "line-length": 120
    },
    "ignore": [
        "node_modules/**",
        "build/**",
        "*.test.csc"
    ]
}
```

### 集成到构建流程

```bash
# 在 CI/CD 中使用
#!/bin/bash

# 运行代码分析
covanalysis analyze ./src --format=json --output=report.json

# 检查是否有错误
if grep -q '"severity": "error"' report.json; then
    echo "代码分析发现错误，构建失败"
    exit 1
fi

echo "代码分析通过"
```

### 实际应用：代码审查助手

```covscript
import covanalysis

class CodeReviewer
    var analyzer = null
    var rules = new list
    
    function construct()
        this.analyzer = covanalysis.Analyzer()
        this.setupRules()
    end
    
    function setupRules()
        # 启用所有推荐规则
        this.analyzer.enableRule("all-recommended")
        
        # 设置严格程度
        this.analyzer.setStrictness("high")
    end
    
    function reviewFile(filename)
        system.out.println("正在审查: " + filename)
        
        var results = this.analyzer.analyzeFile(filename)
        
        var errors = 0
        var warnings = 0
        
        foreach issue in results
            if issue["severity"] == "error"
                errors += 1
            else if issue["severity"] == "warning"
                warnings += 1
            end
            
            this.printIssue(issue)
        end
        
        system.out.println("\n审查完成:")
        system.out.println("  错误: " + to_string(errors))
        system.out.println("  警告: " + to_string(warnings))
        
        return errors == 0
    end
    
    function printIssue(issue)
        var prefix = "[" + issue["severity"] + "] "
        system.out.println(prefix + issue["message"])
        system.out.println("  位置: 第 " + to_string(issue["line"]) + " 行")
        
        if issue.exist("suggestion")
            system.out.println("  建议: " + issue["suggestion"])
        end
    end
    
    function reviewProject(directory)
        var files = system.path.scan(directory)
        var totalErrors = 0
        var totalWarnings = 0
        
        foreach file in files
            if file.ends_with(".csc") || file.ends_with(".ecs")
                var fullPath = directory + "/" + file
                if this.reviewFile(fullPath)
                    system.out.println("✓ " + file + " 通过审查\n")
                else
                    system.out.println("✗ " + file + " 有错误\n")
                end
            end
        end
    end
end

# 使用代码审查助手
var reviewer = new CodeReviewer{}
reviewer.reviewProject("./src")
```

## 2.14.9 argparse - 命令行参数解析

`argparse` 是一个功能强大的命令行参数解析库，可以轻松处理复杂的命令行选项。

### 基本使用

```covscript
import argparse

# 创建解析器
var parser = new argparse.ArgumentParser
parser.program_name = "myapp"
parser.description = "这是一个示例程序"

# 添加位置参数
parser.add_argument("input", true, "输入文件路径")
parser.add_argument("output", false, "输出文件路径")

# 添加选项
parser.add_option("--verbose", true, false, "显示详细信息")
parser.add_option("--format", false, true, "输出格式")

# 设置别名
parser.set_option_alias("--verbose", "-v")
parser.set_option_alias("--format", "-f")

# 设置默认值
parser.set_defaults("output", "output.txt")
parser.set_defaults("--format", "json")

# 解析命令行参数
try
    var args = parser.parse_args(context.cmd_args)
    
    system.out.println("输入文件: " + args.input)
    system.out.println("输出文件: " + args.output)
    system.out.println("详细模式: " + to_string(args.verbose))
    system.out.println("输出格式: " + args.format)
    
catch e
    system.out.println("参数解析错误: " + e.what())
    parser.print_help()
    system.exit(1)
end
```

### 完整示例：文件处理工具

```covscript
import argparse

class FileProcessor
    var verbose = false
    var format = "text"
    
    function process(input_file, output_file)
        if this.verbose
            system.out.println("处理文件: " + input_file)
        end
        
        # 读取输入文件
        var in_file = iostream.fstream(input_file, iostream.openmode.in)
        var content = new string
        
        loop
            var line = in_file.getline()
            if in_file.eof()
                break
            end
            content += line + "\n"
        end
        in_file.close()
        
        if this.verbose
            system.out.println("读取了 " + to_string(content.size) + " 字节")
        end
        
        # 根据格式处理
        var processed = null
        switch this.format
            case "upper"
                processed = content.toupper()
            end
            case "lower"
                processed = content.tolower()
            end
            default
                processed = content
            end
        end
        
        # 写入输出文件
        var out_file = iostream.fstream(output_file, iostream.openmode.out)
        out_file.print(processed)
        out_file.close()
        
        if this.verbose
            system.out.println("输出到: " + output_file)
        end
    end
end

# 主程序
function main(args)
    var parser = new argparse.ArgumentParser
    parser.program_name = "fileproc"
    parser.description = "文件处理工具 - 转换文本格式"
    
    parser.add_argument("input", true, "输入文件路径")
    parser.add_argument("output", false, "输出文件路径")
    
    parser.add_option("--verbose", true, false, "显示详细处理信息")
    parser.set_option_alias("--verbose", "-v")
    
    parser.add_option("--format", false, false, "转换格式 (text/upper/lower)")
    parser.set_option_alias("--format", "-f")
    parser.set_defaults("--format", "text")
    
    parser.add_option("--output", false, false, "输出文件路径")
    parser.set_option_alias("--output", "-o")
    
    try
        var parsed = parser.parse_args(args)
        
        var processor = new FileProcessor
        processor.verbose = parsed.verbose
        processor.format = parsed.format
        
        var output = parsed.output
        if output is null
            output = parsed.input + ".out"
        end
        
        processor.process(parsed.input, output)
        
        system.out.println("处理完成！")
        
    catch e
        system.out.println("错误: " + e.what())
        system.exit(1)
    end
end

# 运行主程序
main(context.cmd_args)
```

使用方式：
```bash
# 显示帮助
cs fileproc.csc -h

# 基本使用
cs fileproc.csc input.txt

# 指定输出和格式
cs fileproc.csc input.txt -o output.txt -f upper

# 详细模式
cs fileproc.csc input.txt -v --format=lower
```

## 2.14.10 其他实用库

### regex - 正则表达式

```covscript
import regex

# 创建正则表达式
var pattern = regex.compile("[0-9]+")

# 匹配
var text = "我有 123 个苹果和 456 个橙子"
var matches = pattern.find_all(text)

foreach match in matches
    system.out.println("找到数字: " + match)
end

# 替换
var result = pattern.replace_all(text, "***")
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
