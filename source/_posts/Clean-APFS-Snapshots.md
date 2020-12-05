---
title: 清理 APFS 快照的shell脚本
date: 2020-11-29 23:26:03
tags: [OS X & macOS, APFS, shell]
thumbnail: https://image.blog.chaosjohn.com/Clean-APFS-Snapshots/DiskUtility.png
---

[欢迎转载，但请在开头或结尾注明原文出处【blog.chaosjohn.com】](https://blog.chaosjohn.com/Clean-APFS-Snapshots.html)

## 背景
`macOS` 从 `10.13 High Sierra`开始，引入了 `APFS(Apple File System)` 替代原先的 `HFS+` 作为默认磁盘格式。

笔者觉得最大的特色在于 `写时拷贝(Copy-on-Write)` 和 `快照(Snapshots)`。对于前者，以后有机会笔者再写文阐述，本文主要针对后者。

快照的引入，可以方便并且快速地恢复到过去的某个时间节点，它与 `时间机器(Time Machine)` 配合起来，可以极大程度的保证数据安全。

但快照也导致一个不良后果，即占据了大量的磁盘空间。比如明明没使用很多文件，怎么磁盘空间耗去大半。

## 查看快照
这里利用到 `tmutil` 这个命令，通过查询 `man` 手册，发现它全称为 `Time Machine utility`，即原用于时间机器。

>$ tmutil listlocalsnapshots /

```
Snapshots for volume group containing disk /:
com.apple.TimeMachine.2020-11-29-004329.local
com.apple.TimeMachine.2020-11-29-014637.local
com.apple.TimeMachine.2020-11-29-024241.local
com.apple.TimeMachine.2020-11-29-034330.local
com.apple.TimeMachine.2020-11-29-044145.local
com.apple.TimeMachine.2020-11-29-054542.local
com.apple.TimeMachine.2020-11-29-064456.local
com.apple.TimeMachine.2020-11-29-074846.local
com.apple.TimeMachine.2020-11-29-084356.local
com.apple.TimeMachine.2020-11-29-094633.local
com.apple.TimeMachine.2020-11-29-104339.local
com.apple.TimeMachine.2020-11-29-114752.local
com.apple.TimeMachine.2020-11-29-134250.local
com.apple.TimeMachine.2020-11-29-144214.local
com.apple.TimeMachine.2020-11-29-154417.local
com.apple.TimeMachine.2020-11-29-164352.local
com.apple.TimeMachine.2020-11-29-174500.local
com.apple.TimeMachine.2020-11-29-184606.local
com.apple.TimeMachine.2020-11-29-194225.local
com.apple.TimeMachine.2020-11-29-204725.local
com.apple.TimeMachine.2020-11-29-214338.local
com.apple.TimeMachine.2020-11-29-224417.local
com.apple.TimeMachine.2020-11-29-234647.local
```

通过 `df -th` 查看磁盘剩余 *153GB*

## 清理快照
这里祭出一行shell脚本
>$ for snapshot in $(tmutil listlocalsnapshots / | awk -F. '{print $4}'); do tmutil deletelocalsnapshots $snapshot; done

```
Deleted local snapshot '2020-11-29-004329'
Deleted local snapshot '2020-11-29-014637'
Deleted local snapshot '2020-11-29-024241'
Deleted local snapshot '2020-11-29-034330'
Deleted local snapshot '2020-11-29-044145'
Deleted local snapshot '2020-11-29-054542'
Deleted local snapshot '2020-11-29-064456'
Deleted local snapshot '2020-11-29-074846'
Deleted local snapshot '2020-11-29-084356'
Deleted local snapshot '2020-11-29-094633'
Deleted local snapshot '2020-11-29-104339'
Deleted local snapshot '2020-11-29-114752'
Deleted local snapshot '2020-11-29-134250'
Deleted local snapshot '2020-11-29-144214'
Deleted local snapshot '2020-11-29-154417'
Deleted local snapshot '2020-11-29-164352'
Deleted local snapshot '2020-11-29-174500'
Deleted local snapshot '2020-11-29-184606'
Deleted local snapshot '2020-11-29-194225'
Deleted local snapshot '2020-11-29-204725'
Deleted local snapshot '2020-11-29-214338'
Deleted local snapshot '2020-11-29-224417'
Deleted local snapshot '2020-11-29-234647'
```
再次通过 `df -th` 查看磁盘剩余 *159GB*，释放出 *6GB* 空间

---

最后，如果该文对读者有些许帮助，考虑下给点捐助鼓励一下呗😊
![](https://image.blog.chaosjohn.com/donate-me.png)
