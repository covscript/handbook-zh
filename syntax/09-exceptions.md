# 2.9 异常处理

异常处理是处理程序运行时错误的重要机制。CovScript 提供了 `throw` 和 `try-catch` 来处理异常。

## 2.9.1 抛出异常（throw）

使用 `throw` 关键字抛出异常。

```covscript
# 抛出字符串异常
function divide(a, b)
    if b == 0
        throw "Division by zero error"
    end
    return a / b
end

# 抛出数字异常
function checkAge(age)
    if age < 0
        throw -1
    end
    if age > 150
        throw -2
    end
    return true
end

# 抛出对象异常
function validateInput(input)
    if input == null
        var error = new hash_map
        error.insert("code", 400)
        error.insert("message", "Input cannot be null")
        throw error
    end
end
```

## 2.9.2 捕获异常（try-catch）

使用 `try-catch` 语句捕获和处理异常。

```covscript
# 基本的 try-catch
function safeDivide(a, b)
    try
        if b == 0
            throw "Division by zero"
        end
        return a / b
    catch e
        system.out.println("Error: " + e)
        return null
    end
end

var result = safeDivide(10, 0)
if result != null
    system.out.println("Result: " + to_string(result))
else
    system.out.println("Division failed")
end
```

### 多层 try-catch

```covscript
# 嵌套的异常处理
function complexOperation()
    try
        # 外层操作
        system.out.println("Starting operation...")
        
        try
            # 内层操作
            var result = riskyOperation()
            return result
        catch inner
            system.out.println("Inner error: " + inner)
            throw "Inner operation failed"
        end
    catch outer
        system.out.println("Outer error: " + outer)
        return null
    end
end
```

### 重新抛出异常

```covscript
function processData(data)
    try
        # 处理数据
        if data == null
            throw "Null data"
        end
        # 更多处理...
    catch e
        system.out.println("Processing error: " + e)
        throw e  # 重新抛出异常
    end
end

function mainProcess()
    try
        var data = getData()
        processData(data)
    catch e
        system.out.println("Main process error: " + e)
    end
end
```

## 2.9.3 异常类型

### 字符串异常

```covscript
function operation1()
    throw "Something went wrong"
end

try
    operation1()
catch e
    system.out.println("Caught string exception: " + e)
end
```

### 数字异常

```covscript
function operation2()
    throw 404  # 错误代码
end

try
    operation2()
catch e
    system.out.println("Caught numeric exception: " + to_string(e))
    if e == 404
        system.out.println("Not found error")
    end
end
```

### 对象异常

```covscript
function operation3()
    var error = new hash_map
    error.insert("code", 500)
    error.insert("message", "Internal server error")
    error.insert("details", "Connection timeout")
    throw error
end

try
    operation3()
catch e
    system.out.println("Error code: " + to_string(e.at("code")))
    system.out.println("Message: " + e.at("message"))
    system.out.println("Details: " + e.at("details"))
end
```

## 2.9.4 自定义异常类

创建自定义异常类以更好地组织错误处理。

```covscript
# 基础异常类
class Exception
    var message = ""
    var code = 0
    
    function construct(msg, c)
        this.message = msg
        this.code = c
    end
    
    function getMessage()
        return this.message
    end
    
    function getCode()
        return this.code
    end
    
    function toString()
        return "[Error " + to_string(this.code) + "] " + this.message
    end
end

# 特定异常类
class FileNotFoundException extends Exception
    var filename = ""
    
    function construct(fname) override
        this.filename = fname
        this.message = "File not found: " + fname
        this.code = 404
    end
    
    function getFilename()
        return this.filename
    end
end

class InvalidArgumentException extends Exception
    var argumentName = ""
    
    function construct(argName, msg) override
        this.argumentName = argName
        this.message = "Invalid argument '" + argName + "': " + msg
        this.code = 400
    end
end

# 使用自定义异常
function openFile(filename)
    if !system.path.exist(filename)
        throw new FileNotFoundException{filename}
    end
    return iostream.fstream(filename, iostream.openmode.in)
end

function processNumber(num)
    if num < 0
        throw new InvalidArgumentException{"num", "must be non-negative"}
    end
    return num * 2
end

# 捕获自定义异常
try
    var file = openFile("nonexistent.txt")
catch e
    if typeid e == typeid new FileNotFoundException{""}
        system.out.println("File error: " + e.getMessage())
        system.out.println("Filename: " + e.getFilename())
    else
        system.out.println("Unknown error: " + e.toString())
    end
end
```

## 2.9.5 异常处理模式

### 资源清理模式

```covscript
function readFile(filename)
    var file = null
    try
        file = iostream.fstream(filename, iostream.openmode.in)
        var content = ""
        
        loop
            var line = file.getline()
            if file.eof()
                break
            end
            content += line + "\n"
        end
        
        return content
    catch e
        system.out.println("Error reading file: " + e)
        return null
    finally
        # 确保文件被关闭（如果支持 finally）
        if file != null
            file.close()
        end
    end
end

# 如果不支持 finally，使用这种模式
function readFileSafe(filename)
    var file = null
    try
        file = iostream.fstream(filename, iostream.openmode.in)
        var content = ""
        
        loop
            var line = file.getline()
            if file.eof()
                break
            end
            content += line + "\n"
        end
        
        file.close()
        return content
    catch e
        if file != null
            file.close()
        end
        system.out.println("Error reading file: " + e)
        return null
    end
end
```

### 错误传播模式

```covscript
# 层级错误处理
function layer3()
    throw "Layer 3 error"
end

function layer2()
    try
        layer3()
    catch e
        system.out.println("Layer 2 caught: " + e)
        throw "Layer 2 error: " + e
    end
end

function layer1()
    try
        layer2()
    catch e
        system.out.println("Layer 1 caught: " + e)
        throw "Layer 1 error: " + e
    end
end

function main()
    try
        layer1()
    catch e
        system.out.println("Main caught: " + e)
    end
end
```

### 错误恢复模式

```covscript
function connectToServer(host, port, maxRetries)
    var retries = 0
    
    loop
        try
            # 尝试连接
            var connection = network.connect(host, port)
            return connection
        catch e
            retries += 1
            if retries >= maxRetries
                system.out.println("Failed after " + to_string(maxRetries) + " retries")
                throw "Connection failed: " + e
            end
            
            system.out.println("Retry " + to_string(retries) + "...")
            runtime.sleep(1000)  # 等待1秒后重试
        end
    end
end

# 使用
try
    var conn = connectToServer("example.com", 8080, 3)
    system.out.println("Connected successfully")
catch e
    system.out.println("Failed to connect: " + e)
end
```

## 2.9.6 实际应用示例

### 文件操作错误处理

```covscript
class FileManager
    function readLines(filename)
        var file = null
        var lines = new list
        
        try
            if !system.path.exist(filename)
                throw new FileNotFoundException{filename}
            end
            
            file = iostream.fstream(filename, iostream.openmode.in)
            
            loop
                var line = file.getline()
                if file.eof()
                    break
                end
                lines.push_back(line)
            end
            
            file.close()
            return lines
        catch e
            if file != null
                file.close()
            end
            
            system.out.println("Error: " + e.toString())
            return null
        end
    end
    
    function writeLines(filename, lines)
        var file = null
        
        try
            file = iostream.fstream(filename, iostream.openmode.out)
            
            foreach line in lines
                file.println(line)
            end
            
            file.close()
            return true
        catch e
            if file != null
                file.close()
            end
            
            system.out.println("Error writing file: " + e)
            return false
        end
    end
end
```

### 网络请求错误处理

```covscript
class HttpClient
    function get(url)
        try
            # 验证 URL
            if url == null || url.empty
                throw new InvalidArgumentException{"url", "URL cannot be empty"}
            end
            
            # 发送请求
            var response = network.http.get(url)
            
            # 检查状态码
            if response.status != 200
                throw "HTTP Error " + to_string(response.status)
            end
            
            return response.body
        catch e
            system.out.println("Request failed: " + e)
            return null
        end
    end
    
    function getWithRetry(url, maxRetries)
        var attempts = 0
        
        loop
            try
                return this.get(url)
            catch e
                attempts += 1
                if attempts >= maxRetries
                    throw "Failed after " + to_string(maxRetries) + " attempts: " + e
                end
                
                system.out.println("Retry " + to_string(attempts) + "...")
                runtime.sleep(1000)
            end
        end
    end
end
```

### 数据验证错误处理

```covscript
class Validator
    function validateEmail(email)
        if email == null || email.empty
            throw new InvalidArgumentException{"email", "cannot be empty"}
        end
        
        if !email.contains("@")
            throw new InvalidArgumentException{"email", "invalid format"}
        end
        
        return true
    end
    
    function validateAge(age)
        if age < 0
            throw new InvalidArgumentException{"age", "cannot be negative"}
        end
        
        if age > 150
            throw new InvalidArgumentException{"age", "unrealistic value"}
        end
        
        return true
    end
    
    function validateUser(userData)
        try
            this.validateEmail(userData.at("email"))
            this.validateAge(userData.at("age"))
            return true
        catch e
            system.out.println("Validation error: " + e.getMessage())
            return false
        end
    end
end

# 使用验证器
var validator = new Validator
var user = new hash_map
user.insert("email", "user@example.com")
user.insert("age", 25)

if validator.validateUser(user)
    system.out.println("User data is valid")
else
    system.out.println("User data is invalid")
end
```

## 异常处理最佳实践

1. **尽早失败**：发现错误立即抛出异常
2. **异常信息要详细**：包含足够的上下文信息
3. **使用自定义异常类**：便于分类处理不同类型的错误
4. **不要捕获后忽略**：捕获异常后要适当处理
5. **资源清理**：确保在异常发生时释放资源
6. **避免过度使用异常**：异常用于异常情况，不是正常流程控制

```covscript
# 好的实践
function processItem(item)
    # 参数验证
    if item == null
        throw new InvalidArgumentException{"item", "cannot be null"}
    end
    
    # 资源管理
    var resource = null
    try
        resource = acquireResource()
        # 处理逻辑
        var result = performOperation(item, resource)
        resource.release()
        return result
    catch e
        # 清理资源
        if resource != null
            resource.release()
        end
        # 记录错误
        system.out.println("Processing failed: " + e)
        # 重新抛出或返回错误
        throw e
    end
end

# 避免的做法
function badPractice()
    try
        # 某些操作
    catch e
        # 什么都不做，吞掉异常
    end
end
```
