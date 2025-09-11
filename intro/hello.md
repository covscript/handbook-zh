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

## CovScript 代码风格

CovScript 是一种基于行对语句进行分割的语言，也就是在不进行特殊声明的情况下将换行（即`'\n'`）作为语句的结尾。

在 CovScript 3 中，如果这样编写代码：
```
var a = 1 +
        2
```
会导致报错。如果确实需要这样编写，则需要显式地使用预处理器指令进行声明：
```
@begin
var a = 1 +
        2
@end
```
CovScript 4 中允许在悬吊符号（Hanging Operators）后面自动忽略换行，如 `+` 的后面。因此在 CovScript 4 中，第一个例子是正确的。
> 因此我们更推荐使用 CovScript 4 编写任何在进行的新项目

而对于代码块（即在 C/C++/C#/Java 等语言中常见的由 `{...}` 包裹的语句序列），CovScript 往往以功能开头、统一的 `end` 结尾，比如：
```
while a > 0
    a = a - 1
end
```
等价于 C/C++ 中的：
```c
while (a > 0)
{
    a = a - 1
}
```
