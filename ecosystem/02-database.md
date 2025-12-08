# 3.2 csdbc - 数据库操作

`csdbc` 提供了统一的数据库访问接口，支持 SQLite、MySQL 等数据库。

### 安装

```bash
cspkg install csdbc
```

### SQLite 基础操作

```covscript
import csdbc_sqlite as csdbc

# 连接到 SQLite 数据库
var db = csdbc.connect("mydata.db")

# 创建表
var create_sql = "CREATE TABLE IF NOT EXISTS users (" +
                "id INTEGER PRIMARY KEY AUTOINCREMENT, " +
                "name TEXT NOT NULL, " +
                "email TEXT UNIQUE, " +
                "age INTEGER)"
db.just_exec(create_sql)

# 插入数据 - 使用预处理语句
var stmt = db.prepare("INSERT INTO users (name, email, age) VALUES (?, ?, ?)")
stmt.bind(1, "Alice")
stmt.bind(2, "alice@example.com")
stmt.bind(3, 25)
stmt.just_exec()

# 批量插入 - 重用预处理语句以提高性能
var users = {
    {"Bob", "bob@example.com", 30},
    {"Charlie", "charlie@example.com", 35}
}

var insert_stmt = db.prepare("INSERT INTO users (name, email, age) VALUES (?, ?, ?)")
foreach user in users
    insert_stmt.bind(1, user[0])
    insert_stmt.bind(2, user[1])
    insert_stmt.bind(3, user[2])
    insert_stmt.just_exec()
end

system.out.println("数据插入成功")

# 查询数据
var query_stmt = db.prepare("SELECT * FROM users")
var result = query_stmt.exec()

system.out.println("用户列表:")
foreach row in result
    system.out.print("ID: " + row[0])
    system.out.print(", 姓名: " + row[1])
    system.out.print(", 邮箱: " + row[2])
    system.out.println(", 年龄: " + row[3])
end

# 更新数据
var update_stmt = db.prepare("UPDATE users SET age = ? WHERE name = ?")
update_stmt.bind(1, 26)
update_stmt.bind(2, "Alice")
update_stmt.just_exec()
system.out.println("数据更新成功")

# 删除数据
var delete_stmt = db.prepare("DELETE FROM users WHERE age > ?")
delete_stmt.bind(1, 30)
delete_stmt.just_exec()
system.out.println("数据删除成功")
```

### 事务处理

```covscript
import csdbc_sqlite as csdbc

var db = csdbc.connect("mydata.db")

# 创建账户表
db.just_exec("CREATE TABLE IF NOT EXISTS accounts (id INTEGER PRIMARY KEY, name TEXT, balance INTEGER)")

try
    # 开始事务
    db.just_exec("BEGIN TRANSACTION")
    
    # 执行多个操作
    var stmt1 = db.prepare("INSERT INTO accounts (name, balance) VALUES (?, ?)")
    stmt1.bind(1, "账户A")
    stmt1.bind(2, 1000)
    stmt1.just_exec()
    
    var stmt2 = db.prepare("INSERT INTO accounts (name, balance) VALUES (?, ?)")
    stmt2.bind(1, "账户B")
    stmt2.bind(2, 500)
    stmt2.just_exec()
    
    # 转账操作
    var stmt3 = db.prepare("UPDATE accounts SET balance = balance - ? WHERE name = ?")
    stmt3.bind(1, 200)
    stmt3.bind(2, "账户A")
    stmt3.just_exec()
    
    var stmt4 = db.prepare("UPDATE accounts SET balance = balance + ? WHERE name = ?")
    stmt4.bind(1, 200)
    stmt4.bind(2, "账户B")
    stmt4.just_exec()
    
    # 提交事务
    db.just_exec("COMMIT")
    system.out.println("事务提交成功")
    
catch e
    # 发生错误时回滚
    db.just_exec("ROLLBACK")
    system.out.println("事务回滚: " + to_string(e))
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

