---
title: iCloud同步排除文件/目录
date: 2020-11-27 14:49:16
tags: [OS X & macOS, iCloud]
---

![iCloud Drive][img1]

[欢迎转载，但请在开头或结尾注明原文出处【blog.chaosjohn.com】](https://blog.chaosjohn.com/iCloud-Sync-Exclusion.html)
## 前言
苹果的iCloud服务，是笔者离不开苹果生态的一大重要原因。

但是苹果鸡贼的很，免费用户5GB存储空间轻轻松松就存满了，所以早年刚出付费升级容量的时候，我就购买了最低的付费容量，`20GB/¥6/月`。虽然不多，但是备份iPhone设备（特别是相册），也是够用的。不过后来苹果良心发现，加量不加价，自动给变更成了50GB。

自从 *OS X 10.10 Yosemite* 开始，iCloud开始可以用于在mac上进行文件存储，又自从 *iOS 11* 开始，iCloud Drive被集成到了iPhone/iPad上，任意设备都可访问和编辑存储在iCloud内的文件，至此，文件跨设备共享和同步，就成了苹果生态的一大杀手锏。`iCloud发展历史` 请移步 [维基百科](https://en.wikipedia.org/wiki/ICloud)

以至于当50GB也不够用的时候，我毫不纠结的升级到了 `200GB/¥20/月` 的套餐（还支持家庭组共享容量）。 [iCloud 储存空间方案和定价](https://support.apple.com/zh-cn/HT201238)

## 痛点问题
为了方便在多台mac设备之间切换却又保持工作的连贯，iCloud Drive还被很多人用作同步工程目录。

举个例子，当前端工程放在iCloud内，光一个 `node_modules` 目录，就能占据你几百MB甚至上GB的空间，何其浪费。

![node_modules地狱][img2]

## 解决办法
将类似 `node_modules` 的目录排除在外。一般来说，具备同步功能的工具（比如Syncthing）都会有 `排除目录/文件` 的设置项，但是iCloud偏偏没有。社区内很多用户向苹果提了改进意见，但是苹果官方就是“不听不听，王八念经”的态度。

![][img3]

但也不能说苹果没有提供排除文件/目录的方案，只不过方案比较恶心罢了。很简单，在目录/文件夹尾部加上 `.nosync` 后缀，就不会被iCloud同步了。

喵喵喵？？？需要我改文件名？？？这叫什么事啊摔！！！

没辙，这个时候只能用“软链接”来实现了：
1. `mv node_modules node_module.nosync` 重命名尾部加 `.nosync`
2. `ln -sf node_modules.nosync node_modules`，将重命名后的目录软链接到 `node_modules`，这样 `npm` 还能通过 `node_modules` 目录管理第三方库

当然，也有开源工具来帮你更高效地做这件事，移步[HaoChuan9421/nosync-icloud](https://github.com/HaoChuan9421/nosync-icloud)

---

最后，如果该文对读者有些许帮助，考虑下给点捐助鼓励一下呗😊
![](hello-world/donate-me.png)


[img1]: iCloud-Sync-Exclusion/head-image.png
[img2]: iCloud-Sync-Exclusion/npm-hell.jpg
[img3]: iCloud-Sync-Exclusion/buting.png
[img3]: iCloud-Sync-Exclusion/buting.png
