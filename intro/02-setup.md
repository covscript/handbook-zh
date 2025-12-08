# 1.2 设置 CovScript 环境

在安装 CovScript 后，需要进行一些初始化设置以获得最佳的开发体验。

## 1.2.1 初始化 cspkg

cspkg 是 CovScript 的包管理器，类似于 Python 的 pip 或 Node.js 的 npm。首次使用前需要初始化。

打开命令行（macOS 中需要先配置环境变量或从启动台打开 CovScript App），输入以下指令：

```bash
cspkg install --import --yes
```

**这个命令的作用**：
- 初始化 cspkg 的包索引
- 导入官方包仓库
- 准备好包管理环境

## 1.2.2 安装 ECS 引导程序

CovScript 4 (ECS) 是 CovScript 的新版本，提供了更多语言特性。要使用 ECS，需要安装引导程序：

```bash
cspkg install ecs_bootstrap --yes
```

**ECS 的作用**：
- 将 `.ecs` 文件编译为 `.csc` 文件
- 提供增强的语法特性（如包自动生成）
- 向后兼容 CSC（CovScript 3）

## 1.2.3 验证安装

运行以下命令测试是否安装成功：

```bash
cs -v        # CovScript 3 解释器版本
ecs -v       # CovScript 4 编译器版本
cspkg -v     # 包管理器版本
cs_dbg -v    # 调试器版本
```

**预期输出示例**：
```
$ cs -v
Covariant Script Programming Language Interpreter Version 3.x.x
...

$ ecs -v
Enhanced Covariant Script Compiler Version 4.x.x
...
```

如果所有命令都正常输出版本信息，说明环境配置成功！

## 1.2.4 配置环境变量（可选）

虽然安装程序会自动配置环境变量，但如果需要自定义，可以设置以下变量：

### Windows (PowerShell)
```powershell
$env:CS_DEV_PATH = "C:\Program Files\CovScript"
$env:PATH += ";$env:CS_DEV_PATH\bin"
```

### Linux/macOS (Bash/Zsh)
```bash
export CS_DEV_PATH="/usr/share/covscript"
export PATH="$PATH:$CS_DEV_PATH/bin"
```

**环境变量说明**：
- `CS_DEV_PATH`：CovScript 安装根目录
- `PATH`：包含 CovScript 可执行文件的路径

## 1.2.5 安装常用扩展库（推荐）

根据开发需求，可以安装以下常用扩展库：

```bash
# 网络编程
cspkg install network --yes

# 数据库支持（SQLite）
cspkg install csdbc_sqlite --yes

# GUI 编程（ImGui）
cspkg install imgui --yes

# 数据分析
cspkg install covanalysis --yes

# 命令行参数解析
cspkg install argparse --yes
```

查看已安装的包：
```bash
cspkg list
```

## 1.2.6 IDE 和编辑器支持

### Visual Studio Code（推荐）

CovScript 官方提供了 VSCode 扩展，支持语法高亮、代码补全和错误提示。

**安装步骤**：
1. 打开 VSCode
2. 进入扩展商店（快捷键：`Ctrl+Shift+X` 或 `Cmd+Shift+X`）
3. 搜索 "CovScript"
4. 安装由 `mikecovlee` 开发的 "The CovScript 3/4 language support for VSCode"

**扩展特性**：
- 语法高亮
- 代码片段
- 基本的代码补全
- 错误诊断

### 其他编辑器

对于 Vim、Emacs、Sublime Text 等编辑器，可以手动配置语法高亮。CovScript 的语法与 C/C++ 类似，可以使用 C 语言的语法高亮作为临时方案。

## 1.2.7 项目结构建议

创建 CovScript 项目时，建议使用以下目录结构：

```
my-project/
├── src/              # 源代码
│   ├── main.csc
│   └── utils.csp
├── tests/            # 测试文件
│   └── test_main.csc
├── docs/             # 文档
│   └── README.md
└── package.json      # 包配置（可选）
```

## 1.2.8 更新 CovScript

要更新到最新版本，请：

1. 从官网或 GitHub Releases 下载最新的安装包
2. 运行安装程序（会自动覆盖旧版本）
3. 更新已安装的包：
```bash
cspkg update --all
```

## 下一步

环境配置完成！现在可以继续学习 [1.3 你好，世界！](./03-hello.md) 编写你的第一个 CovScript 程序。
