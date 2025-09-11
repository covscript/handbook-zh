# 1.3 你好，世界！

## 1.3.1 Preliminaries
在开始之前，我们还需要介绍一下如何编写 CovScript 代码

与大多数编程语言相似，我们首先需要将代码保存在一个文本文件中才能解释或编译

CovScript 的源代码有如下几种类型：
 + `*.csc`: CovScript 3 源文件
 + `*.csp`: CovScript 3 软件包
 + `*.ecs`: CovScript 4 源文件

其中，`*.csc` 和 `*.csp` 能直接运行，无需编译；`*.ecs` 需要编译成 `*.csc` 或 `*.csp` 后才能运行

还有两种特殊的文件格式：
 + `*.csym`: CovScript 3 Debug Symbolics，在 ECS 中用于给 CSC 解释器提供 ECS 与 CSC 代码之间的对应关系，从而使 CSC 解释器报错时能给到准确的位置
 + `*.ecsx`: CovScript 4 语言扩展，类似一种能在编译期展开的高级宏，能够读取 CovScript 4 的 AST 并对其进行深入的解析，可以调用对应的 API 自定义 AST 的生成行为

关于这两种特殊文件的用法，可以参考 [CovAnalysis](https://github.com/covscript/covanalysis) 框架。
