#### pod install

初始化一个项目

\- 生成xcworkspace文件

\- Podfile.lock

\- Pods目录，里面会有默认的文件



使用：

- 已经在Podfile.lock中的库，不会尝试更新版本(不修改Podfile的情况下)。

* 会更新版本的情况

- 新增或删除的库后，会重新修改Podfile.lock文件。对其他库的版本不会做操作

- 写死的版本号，更改版本后 会更新

- 注意：直接指向某个分支，只有第一次会主动拉取，之后无法更新到最新的代码



#### pod update



1. 更新pod本地仓库

2. 根据Podfile更新第三方库