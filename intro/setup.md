# 1.2 设置 CovScript 环境
打开命令行（macOS 中应打开 CovScript App），输入以下指令
## 设置 CSPKG
```
cspkg install --import --yes
```
## 安装 ECS
```
cspkg install ecs_bootstrap --yes
```
## 测试是否安装成功
```
cs -v
ecs -v
cspkg -v
cs_dbg -v
```
这些命令应输出对应的版本信息
## （推荐）安装 VSCode 插件
CovScript 目前仅针对 VSCode 开发了语言扩展，支持关键字高亮和简单的补全

只需要在 VSCode 扩展商店里搜索 "CovScript"，安装由 `mikecovlee` 开发的 "The CovScript 3/4 language support for VSCode" 即可
