# 3.1 covscript-network - 网络编程

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
        break
    end
    
    # 获取并输出接收到的数据
    var msg = state.get_result()
    system.out.print(msg)
    
    # 如果收到 "quit\r\n"，则退出循环
    if msg == "quit\r\n"
        system.out.println("收到退出指令，关闭连接")
        break
    end
end

# 关闭连接
sock.close()
```

### TCP 服务器（同步模式）

```covscript
import network
using network

# 创建 TCP socket
var sock = new tcp.socket
var acpt = tcp.acceptor(tcp.endpoint_v4(8080))

system.out.println("TCP 服务器启动在端口 8080")

loop
    # 接受客户端连接
    runtime.await(sock.accept, acpt)
    
    # 接收数据
    var data = sock.receive(1024)
    system.out.println("收到: " + data)
    
    # 发送响应
    sock.send("服务器收到: " + data)
    
    # 注意：在实际应用中，应该为每个连接创建新的处理线程或协程
    # 这里简化为顺序处理，每次处理完一个连接后再接受下一个
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

