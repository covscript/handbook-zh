# 叁 · 生态系统与扩展库

CovScript 拥有丰富的扩展库生态系统，提供网络编程、数据库操作、GUI 开发、数据分析等功能。本章介绍 CovScript 生态中的主要库及其使用方法。

**注意：** 本章介绍的扩展库需要通过 `cspkg` 包管理器安装。安装方法请参考各章节的说明。

## 目录

+ [3.1 网络编程 (covscript-network)](./01-network.md)
+ [3.2 数据库操作 (csdbc)](./02-database.md)
+ [3.3 GUI 编程 (covscript-imgui)](./03-imgui.md)
+ [3.4 实体组件系统 (ECS)](./04-ecs.md)
+ [3.5 包管理器 (cspkg)](./05-cspkg.md)
+ [3.6 国密算法 (GMSSL)](./06-gmssl.md)
+ [3.7 现代化界面 (Picasso UI)](./07-picasso.md)
+ [3.8 数据分析 (covanalysis)](./08-analysis.md)
+ [3.9 命令行参数解析 (argparse)](./09-argparse.md)
+ [3.10 其他实用库](./10-utilities.md)
+ [3.11 库开发最佳实践](./11-best-practices.md)

## 扩展库生态

CovScript 的扩展库涵盖了多个领域：

### 网络与通信
- **covscript-network**: 异步网络编程、TCP/UDP 通信
- **covscript-gmssl**: 国密算法支持、TLS 安全通信

### 数据处理
- **csdbc**: 统一的数据库访问接口（SQLite、MySQL）
- **covanalysis**: 数据分析框架（DataFrame、CSV、聚合操作）

### 图形界面
- **covscript-imgui**: 即时模式 GUI（调试工具、编辑器）
- **picasso-ui**: 声明式 UI 框架（活动与视图）

### 开发工具
- **cspkg**: 包管理器
- **argparse**: 命令行参数解析
- **ecs**: ECS 引导程序

通过合理使用这些扩展库，可以快速开发各种类型的应用程序。
