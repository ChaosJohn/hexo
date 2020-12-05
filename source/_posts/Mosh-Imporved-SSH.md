title: Mosh(Mobile Shell) -- 增强版SSH
date: 2016-05-30 14:25:15
tags: [Dev, Software] 
---
![](https://image.blog.chaosjohn.com/Mosh-Imporved-SSH/mosh.png) 

[欢迎转载，但请在开头或结尾注明原文出处【blog.chaosjohn.com】](https://blog.chaosjohn.com/Mosh-Imporved-SSH.html)

## 开场白
* Mosh，是Mobile Shell的缩写(Mo+Sh)，是一个增强版的SSH，或者套用官方的话来说，`Mosh is a replacement for SSH`。
* SSH相信大家应该都不陌生，我们都用它来进行远程登录Unix-like的操作系统，比如运行着Linux的云服务器。
* SSH有一个很大的弊病，那就是连接会断。比如新建了一个SSH会话，然后突然网断了，SSH就断开连接终止了。
* Mosh就是用来解决这个问题的，它可以“漫游”。网断了，重连，或者从一个WiFi切换到另一个WiFi，然后又切换成数据流量，会话一直都在。
* 另外一个笔者非常喜欢的特点是--本地响应特别快(intelligent local echo)。笔者有好几台国外的服务器，几乎都是美国的。从中国大陆到美国的Ping值一般都超过200ms，有时候甚至在400ms以上，在这种情况下对比SSH与Mosh的表现：
	* SSH：键盘按下每一个字符，比如“a”，远程服务器接收到本地键盘的响应，然后再告诉本地“这边已接到你按下字母a”，这之后，本地的终端才会显示出字母a。假设Ping值是200ms，那么每次按下按键之后，得有400ms的延迟屏幕才有显示，so，打字一卡一卡的很难受，体验很糟糕。
	* Mosh：键盘每按下一个字符，屏幕上立马就会显示，发送给远程服务器的工作则在后台静默执行，打字非常流畅。

## 安装
* Mac OS X 
	* 从pkg软件包安装：[OS X 10.5–10.9](https://mosh.mit.edu/mosh-1.2.5-leopard.pkg) && [OS X 10.9+](https://mosh.mit.edu/mosh-1.2.5.pkg)
	* HomeBrew：`$ brew install mosh`
	* MacPorts：`$ sudo port install mosh`
* Windows - Cygwin: `C:\> setup.exe -q mobile-shell`
* Chrome浏览器：插件[Mosh for Chrome](https://chrome.google.com/webstore/detail/mosh/ooiklbnjmhbcgemelgfhaeaocllobloj)
* Linux
	* Ubuntu/Debian: `$ sudo apt-get install mosh`
	* ArchLinux: `$ sudo pacman -S mosh` 
	* CentOS/Fedaro：`$ sudo yum install mosh` 
* FreeBSD: `$ sudo pkg install net/mosh`
* 编译安装
	* 下载源码包进行
	
	```bash
	$ wget https://mosh.mit.edu/mosh-1.2.5.tar.gz
	$ tar xvzf mosh-1.2.5.tar.gz
	$ cd mosh-1.2.5
	$ ./configure
	$ make
	# make install
	```

	* 从git拉取源码
	
	```bash
	$ git clone https://github.com/mobile-shell/mosh
	$ cd mosh
	$ ./autogen.sh
	$ ./configure
	$ make
	# make install
	```
	
## 使用方法
* 基本上和SSH一模一样，直接`$ mosh server-address`就好了，因为Mosh的本质还是SSH
* 如果远程主机的SSH端口不是22，比如是2022，则需要`$ mosh remotehost --ssh="ssh -p 2022"`
* 一般来说，远程主机的环境要配置成`en_US.UTF-8`，不然会报“mosh requires a UTF-8 locale.”之类的异常，根本用不了。琢磨了好久，研究出一套多平台的解决方案。 
	* Linux
	
	```bash 
	1. 在/etc/environment中追加一句LC_ALL="en_US.UTF-8"
	2.A [Ubuntu/Debian] $ sudo locale-gen en_US.UTF-8  
	2.B [CentOS/Fedaro] $ sudo localedef -v -c -i en_US -f UTF-8 en_US.UTF-8
	3. reboot
	```

	* FreeBSD：只需在`~/.login_conf`中追加
	
	```bash 
	me:\
        :charset=UTF-8:\
        :lang=en_US.UTF-8:\
        :setenv=LC_COLLATE=C:
	```

	
## 踩坑
* 错误`Nothing received from the server on UDP port 60003`：简单来说，Mosh的网络通信由两部分组成，TCP用于SSH会话+UDP用于会话保持。TCP部分的端口就是SSH的端口，而UDP的端口则是60000~61000之间的随机数。该错误的解决方案将UDP端口加入防火墙的白名单`$ sudo iptables -I INPUT 1 -p udp --dport 60000:61000 -j ACCEPT`
* 笔者家里的那台老MacBook作为文件服务器，常年开着不关机，但是从外网通过Mosh访问回去，会出现找不到mosh-server的异常。![](https://image.blog.chaosjohn.com/Mosh-Imporved-SSH/osx-mosh-exception.png)原因是笔者用HomeBrew安装的Mosh，执行文件被默认放在`/usr/local/bin/`下，在高版本的OSX下，因安全方面的考虑，该路径不在系统的初始PATH里，所以会找不到mosh-server。解决方法是“显式指定执行文件的路径”：
```
$ mosh macbook --server=/usr/local/bin/mosh-server
```
	
## 结语
自从两年前发现了这么个利器，在笔者的日常运维中，它已经完全替代了SSH的存在。最后，如果该文对读者有些许帮助，考虑下给点捐助鼓励一下呗😊
![](https://image.blog.chaosjohn.com/donate-me.png)
