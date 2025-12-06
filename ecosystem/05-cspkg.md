# 3.5 cspkg - 包管理工具

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

