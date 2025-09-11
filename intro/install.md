# 1.1 安装 CovScript
CovScript 是一门跨平台的编程语言，对于大多数朋友来说，直接到[官网](https://covscript.org.cn/)或[CSBuild Release](https://github.com/covscript/csbuild/releases)上下载 CovScript Runtime 安装包即可
## 1.1.1 Windows 系统
CovScript 在 Windows 系统中提供了 `*.msi` 安装包，只需按照步骤安装完毕即可使用，安装包能自动配置好所有的系统环境。
建议在安装后重启系统使环境生效。
> 当前仅提供了在Windows 10/11 x86_64上的预编译包。其他用户可使用[按照指引自行编译](https://github.com/covscript/csbuild#setup-build-environment)
> 
> CovScript 解释器本身在 MSVC 上经过测试可以正常编译、使用，但完整工具链目前仅在 MinGW-w64 上经过测试
## 1.1.2 Linux 系统
CovScript 为基于 Debian 的 Linux 发行版中提供了 `*.deb` 安装包，具体我们只在 Ubuntu 24.04/22.04 中进行了测试。对于这部分用户，只需要从官网中下载后使用如下命令安装：
```
sudo dpkg -i covscript-*.deb
```
> 当前仅提供了在x86_64和ARM 64（AARCH64）上的预编译包。其他用户可使用[按照指引自行编译](https://github.com/covscript/csbuild#setup-build-environment)
> 
> CovScript 解释器本身在 MUSL LibC、Bionic 上均经过测试可以正常编译、使用，但完整工具链目前仅在 GLibC 上经过测试
## 1.1.3 macOS 系统
CovScript 为基于 Apple Silicon 的 Mac 提供了 `*.dmg` 安装包，只需按照步骤安装完毕即可使用。但由于 macOS 的限制，需要从启动台中打开 CovScript 应用才能使用相关的命令。若想在全局使用，请在 `.bashrc`、`.zshrc` 等 Shell 配置文件中增加如下命令：
```
# Covariant Script Settings
export CS_DARWIN_PATH="/Applications/CovScript.app/Contents/MacOS/covscript"
export PATH="$PATH:$CS_DARWIN_PATH/bin"
export CS_DEV_PATH="$CS_DARWIN_PATH"
```
> 我们目前已不再维护基于 Intel 的 Mac 的支持，理论上[按照指引自行编译](https://github.com/covscript/csbuild#setup-build-environment)依旧没问题
>
> CovScript 解释器本身和 CovScript SDK 在 MacOS 上生成的都是 Universal Binary，经过测试可以正常编译、使用，但完整工具链目前仅在 Apple Silicon 上经过测试
## 1.1.4 特殊平台支持情况
我们需要社区的支持！虽然 CovScript 解释器和核心库都没有平台依赖，但仍然缺乏对全部平台的兼容性适配和测试
### 基于龙芯的 Linux
官方支持龙芯 3A3000/4000，但尚未有对 LoongArch 的支持。但由于 Linux 兼容性较好，理论上遵循 CSBuild 文档即可正常编译
### 基于 ARM 的 Linux 如树莓派、华为鲲鹏
遵循 CSBuild 文档即可正常编译。目前提供 Ubuntu 22.04 ARM 64 的 `*.deb` 安装包
### 32 位现代 Windows (>= Windows 7)
遵循相关文档（未完善）可正常编译
### 32 位传统 Windows (Windows XP)
https://github.com/covscript-archives/legacy_windows_support
### 基于 ARM 的 Windows
仅测试过解释器的支持
### 类 BSD Unix
[社区报告](https://github.com/covscript/covscript/issues/100)在 FreeBSD 上能够正常编译和运行
### Android
由于 NDK 的限制，CovScript 虽然能正常编译，但所有的 Extension 都无法正常 `import`（Extension 本质上是个 Shared Library）
