# 1.2 设置 CovScript 环境
打开命令行（macOS 中应打开 CovScript App），输入以下指令
## 设置 CSPKG
```bash
# 只需要在第一次安装时运行
cspkg install --import --yes
```
## 安装 ECS
```bash
cspkg install ecs_bootstrap --yes
```
## 测试是否安装成功
```bash
cs -v
ecs -v
cspkg -v
cs_dbg -v
```
这些命令应输出对应的版本信息
## （推荐）安装 VSCode 插件
CovScript 目前仅针对 VSCode 开发了语言扩展，支持关键字高亮、简单的补全和一键运行功能

只需要在 [VSCode 扩展商店](https://marketplace.visualstudio.com/items?itemName=mikecovlee.covscript)里搜索 "CovScript"，安装由 `mikecovlee` 开发的 "CovScript 3/4 language support for VSCode" 即可

## 安装第三方 CovScript 扩展包
cspkg 默认仅检索 CovScript 官方维护的包，若需要检索由第三方维护的包，你可以通过如下的命令将包索引添加到本地：
```bash
cspkg config source --app <URL>
```
添加完毕后，你可以通过下面的命令检查：
```bash
cspkg config source
```
cspkg 不会在本地索引包信息，因此只需要重新运行 `cspkg install` 即可。
> 免责声明：CovScript 解释器和 cspkg 不会对包的内容和功能进行检查，在添加任何第三方包索引时请确保其维护者是可信的。第三方包索引可能会覆盖官方索引中的包。
