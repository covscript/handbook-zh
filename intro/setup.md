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
