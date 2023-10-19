title: HomeBrew -- OSX下的最强软件包管理器
date: 2016-05-26 17:23:57
tags: [OS X & macOS,Software]
---

![](HomeBrew-The-Best-Package-Manager-On-OSX/HomeBrewPoster.jpg)

[欢迎转载，但请在开头或结尾注明原文出处【blog.chaosjohn.com】](https://blog.chaosjohn.com/HomeBrew-The-Best-Package-Manager-On-OSX.html)

## 1. 什么是软件包管理器
软件包管理器（以下简称包管理器），是用来管理系统下面软件和程序的工具，负责它们的搜索、安装、更新升级和卸载。

## 2. 为什么需要包管理器
* 包管理器一般来说是*nix系统(Unix-like: Linux, Mac OS X, FreeBSD等等)独有的。
* Windows的软件生态系统决定了它不需要包管理器，大部分用户都是用它的图形界面，他们所需要做的就是在获取到该软件的安装包(Installer)后，运行后"Next"->"Next"->...->"Finish"，直至安装到某个路径下，创建软件的启动器快捷方式。<!--软件和软件之间在很大程度上是相互独立的（不是绝对），比如软件A用到了某个库x，软件B也用到了库x，但是他们都在自己的安装路径下存放了库x。（于是，“我的C盘怎么又满了“~~~偷笑）-->
* *nix系统中除了Mac OS X(以下简称OSX)外，其他的系统的主要用途是作为服务器，用Linux做桌面系统来用的家伙不会很多，大部分都是像我这样的苦逼程序猿，用FreeBSD当桌面使用的更是少之又少。那么作为服务器来用的话，一般都不配备图形界面的，只有命令行接口。原因有二：一、图形界面耗性能啊；二、一般服务器都架在云端，你开个图形界面远程登录上去，鼠标拖动都一卡一卡的，那画面，真是太美不忍看。所以一般的Linux的发行版和FreeBSD都提供了包管理器，让用户能在命令行下比较轻松地管理软件（Debian: apt-get, Redhat/CentOS/Fedora: yum, ArchLinux: pacman）。。。呃，那些热衷于任何软件都要从源代码安装的家伙，有毒，我服！
* 不知道是谁说过来着：“MacBook的死忠用户主要有两类：搞设计的和搞IT的”。。。既然搞IT，那肯定离不开命令行。对于我们程序猿来说，OSX真是一个绝妙的组合，既有非常美丽易用的GUI，又有纯*nix的命令行。像ngnix和apache等等这些命令行软件，OSX“出厂”时是没有的，所以要么从下载它们的源代码编译安装，要么就用包管理器来安装。

## 3. OSX下有哪些包管理器 
* __MacPorts__：源于BSD的ports，纯源码编译安装。用简单的话来说，它帮你计算目标软件所需要的依赖，然后把所有依赖和目标软件依次下载源码编译安装。它的优点很明显，软件不依赖操作系统本身，升级系统版本一般不会破坏软件体系；同时缺点也很明显，无论安装还是更新软件，编译源代码实在是太费时间了，笔者的第一台Macbook是09年的经典小白mc207，酷睿P7550的CPU，每次升级MacPorts都只敢在晚上睡觉时进行。MacPorts是笔者最早使用的，陪伴了将近三年时间。最后放弃的原因不是编译费时，而是系统升级到了Yosemite和El Capitan后，有很多软件死活编译不通过，无法更新。
* __Fink__: 很抱歉笔者没有用过，所以在这里我只放科普贴的简介`Fink是基于Debian的packaging tools开发的。最大的特点是安装软件是预编译好的(pre-compiled/pre-built)。 
所以，用Fink安装package是不需要在本机编译的，都是现成的binary code。 Fink最大的问题是package跟进不够快。很多最新版的软件，你要等Fink。` 
* __HomeBrew__: 哈哈哈，有些读者都要怪我啰嗦了，这么半天才开始文章的主要角色。HomeBrew是基于Ruby语言写的，从包管理的特色上来看，它似乎是Macports和Fink俩生的儿子 😂。在依赖处理上，它尽量使用系统原有的库，如果系统不提供，它才安装依赖；安装时优先选择已编译好的二进制包，如果没有它才下载源码进行编译安装。

## 4. 安装HomeBrew 
* 笔者之前介绍过，HomeBrew是用Ruby写的，而OSX内置了Ruby，所以打开终端(Terminal)，把`/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`复制粘贴进去，回车执行，That's all。

## 5. HomeBrew基础命令
* 笔者在这边以安装wget举例（OSX居然没有集成wget，真有点不爽）
* `brew search wget` 搜索软件，列出所有包含目标名称的软件 
* `brew install wget` 安装软件，默认安装到/usr/local下面去
* `brew info wget` 查看安装包信息
* `brew uninstall wget` 卸载软件 
* `brew update` 更新所有软件列表（HomeBrew使用git，直接从github上pull下来）
* `brew upgrade` 升级所有软件 
* `brew cleanup` 清理
* `brew list` 列出所有通过HomeBrew安装的软件 
* `brew doctor`  检查HomeBrew是否存在问题

## 6. 笔者用HomeBrew安装的软件推荐列表
* `wget` 不再赘述
* `htop` top的高级版，更直观，是一款用来监测系统的CPU、内存、uptime、进程等系统信息的工具
* `tree` 以树状形式列出文件目录树
* `tmux` 终端复用软件，神器，谁用谁知道
* `hardlink-osx` 让你给目录加硬链接，对，你没听错，目录！
* `macvim` vim在OSX下的GUI版本，做了很多OSX的本地化，比如cmd+w关闭等等
* `mpg123` `mpg321` 名字很秀逗的两个软件，命令行音乐播放器，可以用来装个逼😀
* `mplayer` `mpv` 两个非常棒的多媒体播放器，个人更偏好后者
* `mtr` traceroute的加强版
* `nvm` 对node进行多版本管理
* `python3` OSX下安装Python3最为方便的方法了

## 7. Cask -- HomeBrew的大杀器
* 一句话介绍：用来安装各种OSX应用程序的工具，例如Chrome, Firefox，MysqlWorkbench。
* 安装Cask：`brew tap phinze/homebrew-cask && brew install brew-cask`
* 使用（以Chrome举例）： 
	* `brew cask search chrome` 搜索应用，结果显示我们要安装google-chrome
	* `brew cask install google-chrome` 安装应用
	* `brew cask uninstall google-chrome` 卸载应用
	* `brew update` 更新所有应用列表
	* `brew cleanup` 清理 
* Cask相比brew缺少了upgrade命令，所以这边奉上一行shell脚本，用来升级所有用Cask安装的应用程序 `for c in $(brew cask list); do ! brew cask info $c | grep -qF "Not installed" || brew cask install $c; done`

## 8. 结语
HomeBrew以给我节省了很多找软件和安装软件的时间，导致我对它越来越喜欢，以至于，我系统里一切能用HomeBrew搜索到的软件，我全用它来搞定了。关于这篇博文，我觉得对某些人来说，那句给Cask进行upgrade的shell脚本，是最具有价值的。（那就~考虑给我点捐助进行鼓励吧，一元两元不嫌少，一百两百不嫌多）
![](hello-world/donate-me.png)
