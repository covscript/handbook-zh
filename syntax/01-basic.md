# 2.1 基础语法

本节介绍 CovScript 的基础语法规则，包括注释、变量声明、常量声明等核心概念。

如果你还没有安装和配置 CovScript，请先阅读 [**壹 · 引言**](../intro/README.md) 章节。

## 2.1.1 注释

CovScript 使用 `#` 符号表示单行注释，从 `#` 开始到行尾的内容都会被忽略。

```covscript
# 这是一个单行注释
var x = 10  # 变量声明后的注释

# 注释可以用来说明代码的功能
# 多行注释需要每行都加 # 符号
```

## 2.1.2 变量声明

使用 `var` 关键字声明变量。CovScript 是动态类型语言，变量在声明时不需要指定类型。

```covscript
# 基本变量声明
var x = 10
var name = "CovScript"
var flag = true

# 变量可以重新赋值
var count = 0
count = 1
count = "now a string"  # 类型可以改变
```

## 2.1.3 常量声明

使用 `constant` 关键字声明常量。常量一旦赋值就不能再修改。

```covscript
# 声明常量
constant PI = 3.14159
constant MAX_SIZE = 100
constant APP_NAME = "MyApp"

# 尝试修改常量会导致错误
# PI = 3.14  # 错误！常量不能修改
```

## 2.1.4 多变量声明

CovScript 支持在一行中声明多个变量。

```covscript
# 同时声明多个变量
var a = 1, b = 2, c = 3

# 混合声明变量和常量
var x = 10
constant y = 20
var z = 30
```

## 2.1.5 语句结束规则

CovScript 中，每个语句通常独占一行，不需要使用分号结束。不过也可以使用分号主动结束语句。

```covscript
# 每行一个语句（推荐）
var x = 10
var y = 20
var sum = x + y

# 使用分号结束语句（可选）
var a = 1; var b = 2; var c = 3
```

**注意事项：**
- 变量名必须以字母或下划线开头
- 变量名可以包含字母、数字和下划线
- 变量名区分大小写
- 不能使用 CovScript 的关键字作为变量名

```covscript
# 合法的变量名
var myVariable = 1
var _private = 2
var data2 = 3
var userName = "Alice"

# 不合法的变量名示例
# var 2data = 1      # 不能以数字开头
# var my-var = 2     # 不能包含连字符
# var function = 3   # 不能使用关键字（如 function, var, if 等）
```
