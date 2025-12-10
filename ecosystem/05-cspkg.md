# 3.5 cspkg - 包管理工具

`cspkg` 是 CovScript 的包管理器，用于安装、更新和管理扩展库。类似于 Python 的 pip、Node.js 的 npm 或 Rust 的 cargo。

## 3.5.1 基本命令

### 安装包

```bash
# 安装包
cspkg install <package_name>

# 安装包并自动确认
cspkg install <package_name> --yes

# 安装特定版本
cspkg install <package_name>@<version>

# 从 Git 仓库安装
cspkg install <package_name> --git <repo_url>

# 初始化 cspkg（首次使用）
cspkg install --import --yes
```

### 管理包

```bash
# 更新包
cspkg update <package_name>

# 更新所有包
cspkg update --all

# 卸载包
cspkg uninstall <package_name>

# 列出已安装的包
cspkg list

# 查看包信息
cspkg info <package_name>

# 搜索包
cspkg search <keyword>
```

### 实用示例

```bash
# 安装常用扩展库
cspkg install network --yes           # 网络编程
cspkg install csdbc_sqlite --yes      # SQLite 数据库
cspkg install imgui --yes             # GUI 编程
cspkg install argparse --yes          # 命令行参数解析
cspkg install covanalysis --yes       # 数据分析

# 安装 ECS 引导程序
cspkg install ecs_bootstrap --yes

# 查看已安装的包
cspkg list

# 更新所有包到最新版本
cspkg update --all
```

## 3.5.2 创建自己的包

### 包的目录结构

推荐的包目录结构：

```
my-package/
├── package.json      # 包配置文件（可选，但推荐）
├── main.csc          # 主入口文件
├── README.md         # 包说明文档
├── LICENSE           # 许可证文件
├── examples/         # 示例代码
│   └── demo.csc
├── tests/            # 测试代码
│   └── test_main.csc
└── src/              # 源代码目录
    ├── module1.csc
    └── module2.csp
```

### 创建 package.json（可选）

虽然 `package.json` 对于 CovScript 包不是必需的，但如果要发布到官方仓库，建议添加：

```json
{
    "name": "my-package",
    "version": "1.0.0",
    "description": "我的 CovScript 包",
    "author": "Your Name <your.email@example.com>",
    "license": "MIT",
    "main": "main.csc",
    "dependencies": {
        "network": "^1.0.0",
        "argparse": "^1.0.0"
    },
    "keywords": ["network", "tool", "utility"],
    "repository": {
        "type": "git",
        "url": "https://github.com/username/my-package"
    },
    "homepage": "https://github.com/username/my-package",
    "bugs": {
        "url": "https://github.com/username/my-package/issues"
    }
}
```

### 编写包代码

**main.csc（主入口文件）：**

```covscript
# my-package/main.csc
package my_package

    # 导入依赖
    import network
    
    # 包的版本
    constant VERSION = "1.0.0"
    
    # 包的功能函数
    function greet(name)
        return "Hello, " + name + "!"
    end
    
    function calculate(a, b)
        return a + b
    end
    
    # 工具类
    class MyTool
        var data = null
        
        function construct(initialData)
            this.data = initialData
        end
        
        function process()
            # 处理数据
            return this.data
        end
    end
    
end
```

### 使用本地包

在其他项目中使用你的包：

```covscript
# 导入包（需要在 CS_DEV_PATH 或当前目录下）
import my_package

# 使用包中的功能
system.out.println(my_package.greet("World"))

var result = my_package.calculate(10, 20)
system.out.println("Result: " + to_string(result))

# 使用包中的类
var tool = new my_package.MyTool{"sample data"}
var processed = tool.process()
```

## 3.5.3 发布包

### 准备发布

1. 确保包的所有文件都已完成
2. 编写完整的 README.md 文档
3. 添加许可证文件（LICENSE）
4. 测试包的所有功能

### 发布到 GitHub

```bash
# 初始化 Git 仓库
git init
git add .
git commit -m "Initial commit"

# 推送到 GitHub
git remote add origin https://github.com/username/my-package.git
git push -u origin master
```

### 发布到 cspkg 仓库（官方）

```bash
# 登录 cspkg 账户（如果有）
cspkg login

# 发布包
cspkg publish

# 发布特定版本标签
cspkg publish --tag beta
cspkg publish --tag stable
```

**注意**：目前 cspkg 的官方包仓库由 CovScript 团队维护，社区包需要通过 GitHub 等方式分享。

## 3.5.4 包的依赖管理

### 指定依赖版本

在 `package.json` 中使用语义化版本：

```json
{
    "dependencies": {
        "network": "^1.0.0",      // 兼容 1.x.x 版本
        "imgui": "~2.1.0",        // 兼容 2.1.x 版本
        "csdbc": "1.0.0"          // 精确版本
    }
}
```

### 依赖安装

```bash
# 安装 package.json 中的所有依赖
cspkg install

# 查看依赖树
cspkg list --tree
```

## 3.5.5 常见问题（FAQ）

### Q: 如何安装本地开发的包？

**A**: 将包目录放在 `$CS_DEV_PATH/imports` 或项目目录下，然后使用 `import` 导入。

### Q: 包安装失败怎么办？

**A**: 
1. 检查网络连接
2. 运行 `cspkg install --import --yes` 重新初始化
3. 查看错误日志，确认包名是否正确

### Q: 如何更新 cspkg 本身？

**A**: cspkg 随 CovScript 一起更新。下载并安装最新的 CovScript 版本即可。

### Q: 如何卸载不需要的包？

**A**: 使用 `cspkg uninstall <package_name>` 命令。

### Q: 包的安装位置在哪里？

**A**: 包默认安装在 `$CS_DEV_PATH/imports` 目录下。

### Q: 如何创建私有包？

**A**: 在 GitHub 或其他 Git 托管平台创建私有仓库，然后使用：
```bash
cspkg install my_package --git https://github.com/username/my-package.git
```

## 3.5.6 最佳实践

1. **版本管理**：使用语义化版本（Semantic Versioning）
2. **文档完善**：编写清晰的 README 和 API 文档
3. **测试充分**：为包的主要功能编写测试用例
4. **依赖最小化**：只依赖必需的包，避免依赖膨胀
5. **兼容性**：明确说明支持的 CovScript 版本
6. **许可证**：选择合适的开源许可证（如 MIT、Apache 2.0）
7. **示例代码**：提供完整的使用示例

## 参考资料

- **cspkg 源码**：https://github.com/covscript/cspkg
- **官方包仓库**：https://github.com/covscript
- **包开发指南**：[3.10 库开发最佳实践](./11-best-practices.md)
- **示例包**：参考官方扩展库的实现，如 network、imgui 等