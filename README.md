
# CovScript 中文手册

欢迎阅读 CovScript 中文手册！欢迎社区伙伴积极反馈、贡献内容（Issue 或 Pull Request）。


## 版本与标准

本手册依据 CovScript 官方标准：`CSC2511XX`（CovScript 3）与 `ECS2106XX`（CovScript 4）编写。


## 关于 CovScript

<img src="https://github.com/covscript/covscript/raw/master/docs/covariant_script_wide.png" width="50%">

CovScript（全称 Covariant Script，中文名：智锐编程语言）始于 2017 年，是一款跨平台通用型动态语言。

- **官网**：[covscript.org.cn](http://covscript.org.cn/)
- **在线文档**：[manual.covscript.org.cn](https://manual.covscript.org.cn/)
- **GitHub 仓库**：[github.com/covscript](https://github.com/covscript/)


### 版本说明

CovScript 目前有两个主要版本：
- **CovScript 3 (CSC)**：2018 年 12 月发布，主流稳定版
- **CovScript 4 (ECS)**：2022 年中旬公测，持续迭代中

自 2023 年 4 月起，ECS 已初步整合 CSC 主要特性。本手册以 ECS 为主，涉及差异时会显著标注。


### 主要特性

- **跨平台**：支持 Windows、Linux、macOS
- **动态类型**：灵活类型系统，开发高效
- **面向对象**：类、继承、多态等 OOP 特性
- **协程支持**：内置 fiber 协程，便于异步编程
- **丰富生态**：网络、数据库、GUI、数据分析等扩展库
- **包管理器**：cspkg 一键管理依赖
- **C++ 扩展**：支持原生模块开发


## 目录

- [壹 · 引言](./intro/README.md)
  - 安装 CovScript
  - 环境配置
  - 第一个程序
- [贰 · 语法基础](./syntax/README.md)
  - 基础语法、数据类型、运算符
  - 控制流、函数、面向对象
  - 模块系统、内存管理、异常处理
  - 迭代器、标准库、高级特性
  - 异步编程与协程
- [叁 · 生态系统与扩展库](./ecosystem/README.md)
  - 网络编程、数据库操作
  - GUI 编程（ImGui、Picasso）
  - 包管理器、国密算法
  - 数据分析、命令行工具
- [常见问题解答（FAQ）](./FAQ.md)
  - 安装与配置、语法特性
  - 标准库、扩展库
  - 性能优化、开发工具
  - 疑难杂症


## 快速开始

### 安装
请参考 [1.1 安装 CovScript](./intro/01-install.md) 获取详细步骤。

### 第一个程序
新建 `hello.csc` 文件：

```covscript
system.out.println("Hello, World!")
```

在终端运行：

```bash
cs hello.csc
```

更多示例见 [1.3 你好，世界！](./intro/03-hello.md)


## 参考资料

本手册内容均基于以下官方资源编写与验证：

### 核心仓库
- [CovScript 主仓库](https://github.com/covscript/covscript)（解释器与测试用例）
- [CovScript 示例集](https://github.com/covscript/covscript-example)
- [CovScript 4 (ECS)](https://github.com/covscript/ecs)
- [CovScript 构建脚本](https://github.com/covscript/csbuild)
- [CovScript 包管理器](https://github.com/covscript/cspkg)
- [CovScript VSCode 支持](https://github.com/covscript/covscript-vscode)

### 标准库
- [正则与 Unicode](https://github.com/covscript/covscript-regex)
- [cURL 网络库](https://github.com/covscript/covscript-curl)
- [进程库](https://github.com/covscript/covscript-process)
- [编解码器库（base32, base64, json, crc32, md5, sha1, sha256）](https://github.com/covscript/covscript-codec)

### 扩展库
- [标准扩展库](https://github.com/covscript/stdutils)
- [网络扩展库](https://github.com/covscript/covscript-network)
- [数据库支持](https://github.com/covscript/csdbc)
- [声明式 GUI](https://github.com/covscript/covscript-imgui)
- [控制台 GUI](https://github.com/covscript/covscript-darwin)
- [国密算法](https://github.com/covscript/covscript-gmssl)
- [数据分析](https://github.com/covscript/covanalysis)
- [SQLite 数据库](https://github.com/covscript/covscript-sqlite)
- [ODBC 连接件](https://github.com/covscript/covscript-database)
- [Zip 压缩包](https://github.com/covscript/covscript-zip)

### 技术博客
- [CovScript 异步编程](https://unicov.cn/2025/10/10/covscript-asyncio/)
- [CovScript 官方文档](https://manual.covscript.org.cn/)


## 贡献指南

欢迎通过 Pull Request 或 Issue 改进本手册！

贡献须知：
1. 代码示例需实际测试，确保可运行
2. API 引用与官方仓库保持一致
3. 术语统一，格式规范
4. 新增功能需配套完整示例代码


## 更新日志

请查阅 [CHANGELOG.md](./CHANGELOG.md) 获取最新文档变更。


## 许可证

本手册遵循 CovScript 官方许可证，详见 [LICENSE](./LICENSE)。
