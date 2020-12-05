---
title: 端口转发(篇一) - rinetd
date: 2020-11-30 17:09:13
tags: [tool, port-forwarding, network]
thumbnail: https://image.blog.chaosjohn.com/Port-Forwarding-1/banner.png
---

[欢迎转载，但请在开头或结尾注明原文出处【blog.chaosjohn.com】](https://blog.chaosjohn.com/Port-Forwarding-1.html)

## 前言
笔者准备写一个 `端口转发` 系列文章，涉及计多个工具和命令。

本文为篇一，介绍一个大家常用的工具，rinetd。

## 简介A: 端口转发
英文：Port forwarding

将一台 `主机A` 的 `端口x` 转发到另一台 `主机B` 的 `端口y` 并由 `主机B` 提供转发的网络服务。

即通过访问 `主机B:端口y` 来访问部署在 `主机A:端口x` 上的服务。

## 简介B: rinetd
[项目主页](http://www.rinetd.com/) / [Github](https://github.com/boutell/rinetd)

rinetd是为在一个Unix和Linux操作系统中为重定向传输控制协议(TCP)连接的一个工具。

rinetd是单一过程的服务器，它处理任何数量的连接到在配置文件etc/rinetd中指定的地址/端口对。

尽管rinetd使用非闭锁I/O运行作为一个单一过程，它可能重定向很多连接而不对这台机器增加额外的负担。


## 场景
最早接触到这个工具，是几年前购买了阿里云的Redis实例，当初该服务未推出公网连接地址，只能通过内网访问，即只在同一个区域的VPC内可达，在当初的业务环境下，只能从阿里云的ECS访问Redis实例。

这就很恼怒了，在笔者本地的开发环境，也要连接改Redis实例，咋办哩。

于是就在阿里云在线文档里找到了这个工具，我还依稀记得那篇文档里，介绍的是如何在CentOS下载rinetd源码（还提供了tar.gz的链接地址），然后编译安装它。（不过伴随着阿里云Redis实例开始支持公网连接后，该文档已经消失在了历史长河里了。）

## 安装
1. 编译安装（适用于任何Linux环境，但个人不推荐，因为还得手动配置开机自启）
```
$ curl -LO 'http://www.rinetd.com/download/rinetd.tar.gz'
$ tar xvzf rinetd.tar.gz
$ cd rinetd
$ sed -i 's/65536/65535/g' rinetd.c （修改端口范围，否则会报错）
$ make
$ sudo make install
```
（提示：官方下载的源码包无法在macOS下完成编译，报错 `error: implicit declaration of function 'inet_addr' is invalid in C99`）

2. 在 `macOS` 上借助 `Homebrew` 安装
```
$ brew install rinetd
```

3. 在 `Debian / Ubuntu` 上借助 `apt` 安装
```
$ sudo apt update
$ sudo apt install rinetd
```

4. 在 `RHEL / CentOS` 上借助 `yum` 安装

官方源中无rinetd，所以需要先安装三方源
```
$ sudo vim /etc/yum.repos.d/nux-misc.repo

[nux-misc]
name=Nux Misc
baseurl=http://li.nux.ro/download/nux/misc/el6/x86_64/
enabled=0
gpgcheck=1
gpgkey=http://li.nux.ro/download/nux/RPM-GPG-KEY-nux.ro
```

然后再安装rinetd
```
$ sudo yum --enablerepo=nux-misc install rinetd
```

## 配置
配置文件路径为 `/etc/rinetd.conf`，并附上那台转发Redis的配置内容
```
#
# this is the configuration file for rinetd, the internet redirection server
#
# you may specify global allow and deny rules here
# only ip addresses are matched, hostnames cannot be specified here
# the wildcards you may use are * and ?
#
# allow 192.168.2.*
# deny 192.168.2.1?


#
# forwarding rules come here
#
# you may specify allow and deny rules after a specific forwarding rule
# to apply to only that forwarding rule
#
# bindadress    bindport  connectaddress  connectport
0.0.0.0	6379	r-ly25ac35fxxxxx.redis.rds.aliyuncs.com	6379


# logging information
logfile /var/log/rinetd.log

# uncomment the following line if you want web-server style logfile format
# logcommon
```
- `allow` 和 `deny` 是配置 *允许和阻止* 的 *IP地址/IP段*
- 转发规则一行一个，可配置多行 `bindadress  bindport  connectaddress  connectport`，如示例里配置为：监听本网卡地址的6379端口号，并且转发到VPC内网Redis实例的6379端口号。

## 运行和自启
- 如果是通过源码编译安装的，命令行直接执行 `rinetd`；自启的话，得手动编写 `Systemd` 的Unit文件来管理或者通过 `Supervisor` 来管理。
- 如果是 `apt` 或 `yum` 安装的，并且init程序是 `Systemd`
  - 启动服务: `$ sudo systemctl start rinetd`
  - 停止服务: `$ sudo systemctl stop rinetd`
  - 重启服务: `$ sudo systemctl restart rinetd`
  - 开机自启: `$ sudo systemctl enable rinetd`
  - 禁用自启: `$ sudo systemctl disable rinetd`

---

最后，如果该文对读者有些许帮助，考虑下给点捐助鼓励一下呗😊
![](https://image.blog.chaosjohn.com/donate-me.png)
