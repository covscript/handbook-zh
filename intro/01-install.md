# 1.1 安装 CovScript
CovScript 是一门跨平台的编程语言，对于大多数朋友来说，直接到官网上下载 CovScript Runtime 安装包即可
## 1.1.1 Windows 系统
CovScript 在 Windows 系统中提供了 `*.msi` 安装包，只需按照步骤安装完毕即可使用，安装包能自动配置好所有的系统环境。
建议在安装后重启系统使环境生效。
> CovScript 官方未维护对 Windows 10 以下版本、32 位 Windows 和 ARM 版 Windows 的支持，请这部分用户[按照指引自行编译](https://github.com/covscript/csbuild#setup-build-environment)
## 1.1.2 Linux 系统
CovScript 为基于 Debian 的 Linux 发行版中提供了 `*.deb` 安装包，具体我们只在 Ubuntu 20.04 中进行了测试。对于这部分用户，只需要从官网中下载后使用如下命令安装：
```
sudo dpkg -i covscript-*.deb
```
> 对于使用其他发行版或官方编译的 Deb 包不能正常使用的情况，请[按照指引自行编译](https://github.com/covscript/csbuild#setup-build-environment)
## 1.1.3 macOS 系统
CovScript 为基于 Intel 的 Mac 提供了 `*.dmg` 安装包，只需按照步骤安装完毕即可使用。但由于 macOS 的限制，需要从启动台中打开 CovScript 应用才能使用相关的命令。若想在全局使用，请在 `.bashrc`、`.zshrc` 等 Shell 配置文件中增加如下命令：
```
# Covariant Script Settings
export CS_DARWIN_PATH="/Applications/CovScript.app/Contents/MacOS/covscript"
export PATH="$PATH:$CS_DARWIN_PATH/bin"
export CS_DEV_PATH="$CS_DARWIN_PATH"
```
> 尚未有对基于 Apple Silicon 的 Mac 的支持
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