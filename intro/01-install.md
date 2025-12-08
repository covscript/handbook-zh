# 1.1 安装 CovScript

CovScript 是一门跨平台的编程语言，对于大多数朋友来说，直接到官网上下载 CovScript Runtime 安装包即可。

**官方下载地址**：http://covscript.org.cn/download （或通过 GitHub Releases 获取）

**GitHub Releases**：https://github.com/covscript/covscript/releases

## 1.1.1 Windows 系统

CovScript 在 Windows 系统中提供了 `*.msi` 安装包，只需按照步骤安装完毕即可使用，安装包能自动配置好所有的系统环境。

**安装步骤**：
1. 从官网或 GitHub Releases 下载 `.msi` 安装包
2. 双击运行安装程序，按照向导完成安装
3. 安装完成后，建议重启系统使环境变量生效
4. 打开命令行（CMD 或 PowerShell），输入 `cs -v` 验证安装

**环境变量**：安装程序会自动配置以下环境变量：
- `CS_DEV_PATH`：CovScript 安装目录
- `PATH`：添加 CovScript 可执行文件路径

> **注意**：CovScript 官方未维护对 Windows 10 以下版本、32 位 Windows 和 ARM 版 Windows 的支持，请这部分用户[按照指引自行编译](https://github.com/covscript/csbuild#setup-build-environment)
## 1.1.2 Linux 系统

CovScript 为基于 Debian 的 Linux 发行版提供了 `*.deb` 安装包，具体我们只在 Ubuntu 20.04 及以上版本中进行了测试。

**安装步骤**（Debian/Ubuntu）：
1. 从官网或 GitHub Releases 下载 `.deb` 安装包
2. 使用以下命令安装：
```bash
sudo dpkg -i covscript-*.deb
```
3. 如果遇到依赖问题，运行：
```bash
sudo apt-get install -f
```
4. 验证安装：
```bash
cs -v
```

**环境变量**：Deb 包会自动配置环境变量，默认安装路径为 `/usr/share/covscript`。

**其他 Linux 发行版**：
- Arch Linux 用户可以从 AUR 安装
- Fedora/CentOS 用户需要从源代码编译
- 通用方法：使用 [csbuild](https://github.com/covscript/csbuild) 从源码编译

> **注意**：对于使用其他发行版或官方编译的 Deb 包不能正常使用的情况，请[按照指引自行编译](https://github.com/covscript/csbuild#setup-build-environment)
## 1.1.3 macOS 系统

CovScript 为基于 Intel 的 Mac 提供了 `*.dmg` 安装包，现在也支持 Apple Silicon (M1/M2/M3) 的 Mac（通用二进制）。

**安装步骤**：
1. 从官网或 GitHub Releases 下载 `.dmg` 安装包
2. 打开 DMG 文件，将 CovScript.app 拖拽到 Applications 文件夹
3. 首次运行时，需要从启动台打开 CovScript 应用以授权
4. 配置全局命令行访问（可选，推荐）

**配置全局命令行访问**：

在 `.bashrc`、`.zshrc` 等 Shell 配置文件中增加如下命令：

```bash
# Covariant Script Settings
export CS_DARWIN_PATH="/Applications/CovScript.app/Contents/MacOS/covscript"
export PATH="$PATH:$CS_DARWIN_PATH/bin"
export CS_DEV_PATH="$CS_DARWIN_PATH"
```

保存后，运行 `source ~/.zshrc`（或相应的配置文件）使配置生效。

**验证安装**：
```bash
cs -v
ecs -v
cspkg -v
```

> **注意**：目前官方编译的解释器二进制文件是通用二进制（Universal Binary），理论上支持 Apple Silicon，但仍在测试中
## 1.1.4 特殊平台支持情况
我们需要社区的支持！虽然 CovScript 解释器和核心库都没有平台依赖，但仍然缺乏对全部平台的兼容性适配和测试
### 基于龙芯的 Linux
官方支持龙芯 3A3000/4000，但尚未有对 LoongArch 的支持。但由于 Linux 兼容性较好，理论上遵循 CSBuild 文档即可正常编译
### 基于 ARM 的 Linux 如树莓派、华为鲲鹏
遵循 CSBuild 文档即可正常编译
### 32 位现代 Windows (>= Windows 10)
遵循相关文档（未完善）可正常编译
### 32 位传统 Windows (Windows XP)
https://github.com/covscript-archives/legacy_windows_support
### 基于 ARM 的 Windows
仅测试过解释器的支持
### 基于 Apple Silicon 的 macOS
仅测试过解释器的支持，目前官方编译出的解释器二进制文件就是通用二进制 (Universal Binary)