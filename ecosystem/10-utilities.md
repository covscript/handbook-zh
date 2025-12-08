# 3.10 其他实用库

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

