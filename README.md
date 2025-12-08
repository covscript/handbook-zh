# CovScript 中文使用手册

欢迎阅读 CovScript 使用手册！该手册目前使用中文编写，欢迎社区的伙伴们多提意见、提 Pull Request！

Welcome to the CovScript Handbook! This handbook is currently written in Simplified Chinese, welcome comments and Pull Requests from community partners!

## 关于 CovScript

<img src="https://github.com/covscript/covscript/raw/master/docs/covariant_script_wide.png" width="50%">

CovScript，全称 Covariant Script，中文名为*智锐编程语言*，始创于2017年，是一门跨平台的通用型动态语言。

**官网**：http://covscript.org.cn/

**在线文档**：https://csman.info/

**GitHub 主仓库**：https://github.com/covscript/covscript

### 版本说明

CovScript 目前活跃的有两个主要版本，分别是：
 + **CovScript 3 (CSC)**：最早发布于2018年12月，是当前的主流版本
 + **CovScript 4 (ECS)**：于2022年中旬开始公测，当前处于测试状态

到2023年4月，ECS已经初步完成了与CSC的整合，因此该手册主要基于ECS进行展示。对于ECS和CSC有区别的地方，我们会在文中显著说明。

### 主要特性

- **跨平台**：支持 Windows、Linux、macOS 等主流操作系统
- **动态类型**：灵活的类型系统，简化开发流程
- **面向对象**：支持类、继承、多态等面向对象特性
- **协程支持**：内置 fiber 协程，便于异步编程
- **丰富生态**：提供网络、数据库、GUI、数据分析等扩展库
- **包管理器**：使用 cspkg 轻松管理依赖
- **C++ 扩展**：可以通过 C++ 编写原生扩展模块

## 目录

 + [壹 · 引言](./intro/README.md)
   - 安装 CovScript
   - 设置开发环境
   - 第一个程序
 + [贰 · 语法基础](./syntax/README.md)
   - 基础语法、数据类型、运算符
   - 控制流、函数、面向对象编程
   - 模块系统、内存管理、异常处理
   - 迭代器、标准库、高级特性
   - 异步编程与协程
 + [叁 · 生态系统与扩展库](./ecosystem/README.md)
   - 网络编程、数据库操作
   - GUI 编程（ImGui、Picasso）
   - 包管理器、国密算法
   - 数据分析、命令行工具

## 快速开始

### 安装

请参考 [1.1 安装 CovScript](./intro/01-install.md) 获取详细安装说明。

### 第一个程序

创建 `hello.csc` 文件：

```covscript
system.out.println("Hello, World!")
```

运行程序：

```bash
cs hello.csc
```

更多示例请参考 [1.3 你好，世界！](./intro/03-hello.md)

## 参考资料

本手册的内容基于以下官方资源编写和验证：

### 核心仓库
- [CovScript 主仓库](https://github.com/covscript/covscript) - 核心解释器和测试用例
- [CovScript 示例集](https://github.com/covscript/covscript-example) - 官方代码示例
- [CovScript 4 (ECS)](https://github.com/covscript/ecs) - ECS 引导程序

### 扩展库
- [网络模块 (covscript-network)](https://github.com/covscript/covscript-network)
- [数据库支持 (csdbc)](https://github.com/covscript/csdbc)
- [GUI 编程 (covscript-imgui)](https://github.com/covscript/covscript-imgui)
- [国密算法 (covscript-gmssl)](https://github.com/covscript/covscript-gmssl)
- [数据分析 (covanalysis)](https://github.com/covscript/covanalysis)
- [包管理器 (cspkg)](https://github.com/covscript/cspkg)

### 技术博客
- [CovScript asyncio 新特性](https://unicov.cn/2025/10/10/covscript-asyncio/)
- [CovScript 官方文档](https://manual.covscript.org.cn/)

## 贡献指南

欢迎提交 Pull Request 或 Issue 来改进本手册！

在贡献时，请确保：
1. 代码示例经过实际测试，能够正常运行
2. 引用的 API 与官方仓库保持一致
3. 保持文档的术语一致性和格式统一
4. 为新增的功能提供完整的示例代码

## 更新日志

查看 [CHANGELOG.md](./CHANGELOG.md) 了解文档的最新更新。

## 许可证

本手册遵循与 CovScript 相同的许可证。详见 [LICENSE](./LICENSE)。
