# CovScript 中文手册更新日志

## 2025-12-06 - 文档全面检查与完善

本次更新对 CovScript 中文手册进行了全面的检查、审核与完善，所有改进均基于最新的代码仓库和实际示例。

### 新增内容

#### 1. 国密算法支持 (GMSSL)
- **SM3 哈希算法**：完整的哈希、HMAC 和 PBKDF2 示例
- **SM4 对称加密**：CTR 和 CBC 模式的加密解密
- **SM2 非对称加密**：密钥生成、PEM 格式、加密解密、数字签名
- **TLS 实现**：基于 simple_tls.csp 的安全通信完整示例
- **编码工具**：Base64、Hex、字节转换、随机数生成

#### 2. Picasso UI 框架
- 声明式 UI 构建方式
- 活动（Activity）和视图（View）系统
- 待办事项应用完整示例
- 响应式布局和自定义样式
- 事件监听和处理机制

#### 3. 静态代码分析 (covanalysis)
- 代码质量检查工具介绍
- 支持的检查项说明
- 自定义检查规则
- 配置文件使用
- CI/CD 集成示例
- 代码审查助手实现

#### 4. 命令行参数解析 (argparse)
- ArgumentParser 类的完整用法
- 位置参数和选项参数
- 参数别名和默认值
- 帮助信息生成
- 文件处理工具完整示例

### 重大更新

#### 1. 网络模块 (covscript-network)
**变更原因**：基于最新的 netutils.ecs 和测试代码更新

- ✅ **异步网络编程**：添加 async.poll_once、async.state、async.work_guard 示例
- ✅ **正确的 TCP API**：修正 socket、acceptor、endpoint 的使用
- ✅ **UDP 通信**：更新为实际的 API 调用方式
- ✅ **聊天服务器**：基于实际 chat_server.csc 的 UDP 实现
- ✅ **异步读写**：async.read_until、async.write 的正确用法

#### 2. ImGui GUI 编程
**变更原因**：基于 tests/test.csc 实际测试代码

- ✅ **中文字体支持**：imgui_font.source_han_sans 的正确使用
- ✅ **窗口应用创建**：window_application 和主循环
- ✅ **控件使用**：slider_float、progress_bar、radio_button、combo_box
- ✅ **布局控制**：begin_window、set_window_pos、set_window_size
- ✅ **实用示例**：文本编辑器、显示器信息获取
- ✅ **删除过时代码**：移除不存在的 API 调用

#### 3. 数据库操作 (csdbc)
**变更原因**：基于 csdbc.csp 实际接口定义

- ✅ **连接方式**：csdbc_sqlite.connect() 的正确用法
- ✅ **预处理语句**：统一使用 db.prepare()、stmt.bind()、stmt.just_exec()
- ✅ **查询执行**：stmt.exec() 返回结果集
- ✅ **事务管理**：使用 SQL 命令（BEGIN/COMMIT/ROLLBACK）
- ✅ **一致性改进**：所有示例使用相同的 API 风格

### 代码质量改进

1. **API 准确性**
   - 所有网络 API 调用与 covscript-network 仓库一致
   - GMSSL API 与 covscript-gmssl 仓库一致
   - ImGui API 与实际测试用例一致
   - 数据库 API 与 csdbc 包定义一致

2. **示例完整性**
   - 每个功能都有完整可运行的示例
   - 示例代码来自实际仓库中的测试和示例文件
   - 添加了更多实用的完整应用示例

3. **代码审查修复**
   - 修正 TCP/UDP socket 使用混淆
   - 统一数据库事务 API 调用
   - 确保所有示例的内部一致性

### 参考资料来源

本次更新参考了以下仓库的最新代码：

- [covscript/covscript](https://github.com/covscript/covscript) - 主仓库测试用例
- [covscript/covscript-example](https://github.com/covscript/covscript-example) - 官方示例集
- [covscript/covscript-gmssl](https://github.com/covscript/covscript-gmssl) - 国密算法支持
- [covscript/covscript-network](https://github.com/covscript/covscript-network) - 网络模块
- [covscript/covscript-imgui](https://github.com/covscript/covscript-imgui) - ImGui 支持
- [covscript/picasso-ui](https://github.com/covscript/picasso-ui) - Picasso UI 框架
- [covscript/csdbc](https://github.com/covscript/csdbc) - 数据库支持
- [covscript/covanalysis](https://github.com/covscript/covanalysis) - 静态分析工具
- [covscript/ecs](https://github.com/covscript/ecs) - ECS 引导程序
- [covscript/cspkg](https://github.com/covscript/cspkg) - 包管理器

### 文档结构

更新后的文档结构：

```
handbook-zh/
├── README.md
├── CHANGELOG.md (新增)
├── intro/
│   ├── 01-install.md
│   ├── 02-setup.md
│   └── 03-hello.md
└── syntax/
    ├── 01-basic.md
    ├── 02-datatypes.md
    ├── 03-operators.md
    ├── 04-control-flow.md
    ├── 05-functions.md
    ├── 06-oop.md
    ├── 07-modules.md
    ├── 08-memory.md
    ├── 09-exceptions.md
    ├── 10-iterators.md
    ├── 11-stdlib.md
    ├── 12-advanced.md
    ├── 13-asyncio.md (已审核)
    ├── 14-ecosystem.md (大幅更新)
    └── 15-examples.md
```

### 下一步计划

- [ ] 添加更多 .csc/.ecs 示例文件到仓库
- [ ] 补充 ECS (CovScript 4) 特有功能说明
- [ ] 添加性能优化和最佳实践章节
- [ ] 完善标准库函数参考
- [ ] 添加常见问题解答 (FAQ)

---

**贡献者**: GitHub Copilot Workspace Agent  
**审核**: 待审核  
**更新日期**: 2025-12-06
