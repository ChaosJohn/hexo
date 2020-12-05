title: 安卓平板上的开发者实用工具 
date: 2016-05-29 16:52:15
tags: [Software, Utility, Android, Developer]
---
![](https://image.blog.chaosjohn.com/Tools-for-Developers-on-Android-Tablet/my-mipad.jpg)

[欢迎转载，但请在开头或结尾注明原文出处【blog.chaosjohn.com】](https://blog.chaosjohn.com/Tools-for-Developers-on-Android-Tablet.html)

## 1. 测试机型和本文的受众
* 你得有一块android平板，或者一只超大屏的android手机(类似小米MAX，其实小米Note也凑合)。
* 你还得是个开发者，本文介绍的工具都是为开发者准备，如果你不是，请拉到文章末尾，就只给你安利GMD GestureControl这一个软件啦🙈。
* [本文封面](https://image.blog.chaosjohn.com/Tools-for-Developers-on-Android-Tablet/my-mipad.jpg)的是笔者的小米平板第一代64G版，于15年“双十一”花1099块大洋买了送给女朋友的，后来公司年终抽奖抽到一块iPad mini2，就从女朋友手里把这块安卓板子换回来了（笔者是标准的暖男嚯嚯嚯）。
* 笔者绝对没有给小米打广告，相反除了这块MiPad之外，笔者只想狠狠地黑小米，比如小米MAX--这货到底是手机还是平板啊；比如说公司花3000块钱买回来的小米Note顶配版--特喵的一天三充而且果然“为发烧而生”。
* 只有这块14年问世的MiPad，用下来觉得还好，续航很给力，而且“NVIDIA Tegra K1”的CPU，放到现在16年中旬，还能排上号。![](https://image.blog.chaosjohn.com/Tools-for-Developers-on-Android-Tablet/mobile-cpu-score.png)
* 笔者虽然是果粉，但是绝对不鄙视android，相反在各类平板中选择的话，我还是比较偏好android的。原因无他，用的顺手。（以后我会写篇文章讲讲我为什么平板选择android，而手机却选择iPhone）

## 2. 系统版本
* 小米平板嘛，出厂必须自带MIUI滴。
* 想把小米平板变成“PC”，还刷过一次[RemixOS](https://cn.jide.com/remixos)，但是因为官方没有支持MiPad，只有民间大神制作的兼容包，还只有1.0版本的，虽然用着体验挺好的，最后实在架不住某些蛋疼的bug，还是刷回了MIUI系统。关于RemixOS，如果有感兴趣的，可以去官网下载他们的PC版，安装到u盘上直接插电脑体验，不建议刷到大家的平板上。
* 为了Root，所以刷了MIUI的开发版本，已升级到MIUI7(Android 4.4.4)，删除了系统的Updater.apk，禁用了自动更新。

## 3. 必装应用列表
* [谷歌安装器](https://www.gugeanzhuangqi.com)（需root）：给国内被阉割的安卓系统安装“谷歌服务框架”和“谷歌应用市场”。
* ES File Explorer：笔者在安卓平台上用过的最好用的文件管理器，给开发者推荐的原因如下
	- 方便管理已安装应用，还可以备份应用（包括应用数据）
	- 支持ftp/sftp，方便连接本地局域网的电脑或云端的Linux主机，进行传输文件。
* Root Explorer（需root）：俗称“RE管理器”，可以直接将系统路径挂载为可写，还可以原生读取应用的db文件。
* AirDroid：无线传输文件和管理安卓设备。
	- 从电脑上查看安卓设备的内容（照片、收发短信等）![](https://image.blog.chaosjohn.com/Tools-for-Developers-on-Android-Tablet/Airdroid-Web.png)
	- 把电脑当做平板的“键盘”，当电脑上点中Airdroid的窗口后，电脑键盘的输入都会同步到平板上去。![](https://image.blog.chaosjohn.com/Tools-for-Developers-on-Android-Tablet/Airdroid-Keyboard.png)
* Hacker's Keyboard：“黑客键盘”，模拟标准的PC键盘，包括F1~F12，ctrl，alt，shift，tab等功能键。非常实用。![](https://image.blog.chaosjohn.com/Tools-for-Developers-on-Android-Tablet/Hacker's%20Keyboard-A.png)![](https://image.blog.chaosjohn.com/Tools-for-Developers-on-Android-Tablet/Hacker's%20Keyboard-B.png)
* [Serverauditor](https://serverauditor.com)：SSH客户端，配色非常小清新，而且跨平台，在iOS和chrome上都有。还能自动判别所连接的主机是什么系统并且给相应的主机加上对应系统的Logo。![](https://image.blog.chaosjohn.com/Tools-for-Developers-on-Android-Tablet/Serverauditor-home.png) ![](https://image.blog.chaosjohn.com/Tools-for-Developers-on-Android-Tablet/Serverauditor-session.png)
* [ConnectBot](https://github.com/connectbot/connectbot)：也是一款SSH客户端，官方Github的介绍为“ConnectBot is the first SSH client for Android”。功能比Serverauditor强大，而且Hacker's Keyboard专门为其做了优化（详见Hacker's Keyboard的设置页）。![](https://image.blog.chaosjohn.com/Tools-for-Developers-on-Android-Tablet/ConnectBot-GooglePlay.png)
* [IrssiConnectBot](https://dan.drown.org/android/mosh/)：以ConnectBot为原型的SSH客户端，增加了mosh协议。切记，千万不要去GooglePlay商店安装，那里下载的版本和最基础的ConnectBot一毛一样，根本就没有mosh！请到IrssiConnectBot的官网下载或者[点击此处下载](https://dan.drown.org/android/mosh/irssiconnectbot-release.apk)或者去[它的Github](https://github.com/irssiconnectbot/irssiconnectbot)下载源码到本地自己编译。
* [JuiceSSH](https://play.google.com/store/apps/details?id=com.sonelli.juicessh)：看名称就知道它也是一款SSH客户端了，同样支持mosh协议，而且是[mosh官网](https://mosh.mit.edu)推荐的mosh客户端。功能非常强大，还支持插件。但是！笔者不喜欢JuiceSSH，因为笔者远程管理主机一般都会使用到Tmux，需要按缀键Ctrl-B来激活，但是JuiceSSH不识别同时按下的Ctrl键和B键，故抛弃。但不妨碍它成为GooglePlay上最畅销的SSH客户端之一。
* Shadowsocks：不解释，懂得人自然懂，不懂的人，有“病”的话搜索一下（哈哈哈，其实是bing.com），百度该词已被和谐。当然建议从GooglePlay上安装。
* BusyBox（需root）：BusyBox 被称为 Linux 工具里的瑞士军刀.简单的说BusyBox就好像是个大工具箱，它集成压缩了 Linux 的许多工具和命令。Android的本质是Linux，但其底层的一些工具不仅较标准的Linux少很多，已有的几乎也是功能不完善的。BusyBox的作用就是补全和替换掉这些Linux工具。
* [Linux Deploy](https://play.google.com/store/apps/details?id=ru.meefik.linuxdeploy)（需root和安装BusyBox）：最后祭上的终极杀器，[官方Github](https://play.google.com/store/apps/details?id=ru.meefik.linuxdeploy)。简单一句话介绍--“在你的安卓设备上原生跑Linux”。基本原理--“chroot”，想详细了解的可以从这个关键词入手。可以选择的Linux发行版有很多，主流的都有，包括Debian、Ubuntu、CentOS、Arch Linux、Gentoo等等，安装完成后可以通过127.0.0.1:22进行SSH登入。如果平板的空间足够的话，可以同时安装很多个(如下图所示，笔者安装了Arch Linux、Ubuntu和Gentoo)，但只能同时选择一个运行。![](https://image.blog.chaosjohn.com/Tools-for-Developers-on-Android-Tablet/Linux-Deploy.png) ![](https://image.blog.chaosjohn.com/Tools-for-Developers-on-Android-Tablet/Linux-Deploy-Installed.png)
* [GMD GestureControl Lite](https://play.google.com/store/apps/details?id=com.goodmooddroid.gesturecontrol)（需root）：最后的最后，送上的一枚手势软件（并非只为开发者推荐哦）。这边给出的链接是Lite版（非Lite版要价5.55美元），但是Lite版就够用了。羡慕iPad的各种手势吗？四指内抓退回桌面、四指左右横扫切换上一个应用和下一个应用、四指上推显示最近应用列表。。。有了这个软件，这些功能你都能实现！而且实现的更多！简直堪比iOS的越狱插件Activator！![](https://image.blog.chaosjohn.com/Tools-for-Developers-on-Android-Tablet/GMD-GestureControl-Lite.png)

## 4. 结语
其实还有很多很多其他为开发者准备的软件，如果有读者愿意推荐的话，请在下面给我留言啦，小弟不胜感激！最后，如果该文对读者有些许帮助，考虑下给点捐助鼓励一下呗😊
![](https://image.blog.chaosjohn.com/donate-me.png)
