---
title: 端口转发(篇二) - xinetd
date: 2020-12-07 15:50:23
tags: [tool, port-forwarding, network]
thumbnail: https://image.blog.chaosjohn.com/Port-Forwarding-2/banner.png
---

[欢迎转载，但请在开头或结尾注明原文出处【blog.chaosjohn.com】](https://blog.chaosjohn.com/Port-Forwarding-2.html)

这是 `端口转发` 系列文章的第二篇，历史文章：
- [端口转发(篇一) - rinetd](https://blog.chaosjohn.com/Port-Forwarding-1.html)

## 前言
笔者之前也过一篇文 - [IPv6在虚拟机通过无线网卡桥接的网络下无法使用(VMware WorkStation)](https://image.blog.chaosjohn.com/IPv6-not-working-from-bridged-wireless)

文中背景：笔者买了一台 `GK41`，运行的是随机安装的 `Windows 10`，同时借助 `VMware Workstation` 运行了一台虚拟客户机 `Manjaro Linux`。`GK41` 通过 `RJ45` 网口接入网络，虚拟机采用 `桥接` 该网口的有线网卡，给客户机 `Manjaro Linux` 提供网络接入。至此，主机和客户机都处于同一个局域网内，且都能分配到IPv4/IPv6，此时
- `GK41`：192.168.0.41
- `Manjaro Linux`：192.168.0.65

## 诉求
因为笔者不给 `GK41` 接显示器，但又想通过 `IPv6` 远程访问 `Windows 10` 的桌面，结合笔者之前造过 `crontab + ddns6` 的轮子，所以打算：
- `Manjaro Linux` 配置为开机自启
- `Manjaro Linux` 上配置 `ddns6`，通过 `cron` 定时检测并更新自身的 `IPv6地址` 到域名 `manjaro-gk41.example.com`
- `Manjaro Linux` 转发自己的 `3389` 端口到 `Windows 10` 的 `3389` 端口
> `3389` 端口是 `RDP(Remote Desktop Protocal)（微软远程桌面协议）` 的默认端口

## 发现问题
最直接的想到了 [篇一](https://blog.chaosjohn.com/Port-Forwarding-1.html) 中的工具 - `rinetd`

然后就是 *一顿操作猛如虎，试完发现不靠谱*

因为 `rinetd` 不支持 `IPv6`，它只监听 `IPv4地址` 和转发到 `IPv4地址`，详见 `rinetd` 在其 `Github` 上的某 `Issue` - [does the program support ipv6?](https://github.com/boutell/rinetd/issues/2)。

老爷子回复道：可能不会支持 IPv6 了，因为这玩意自从上世纪九十年代开始，我就没怎么动过了，添加 IPv6 支持估计会很难。
![rinetd 作者称不会添加 IPv6 支持](https://image.blog.chaosjohn.com/Port-Forwarding-2/rinetd-will-not-support-ipv6.png)

## 解决问题
在寻找 `rinetd` 替代品的过程中，笔者发现了 `xinetd`

> `xinetd` 和 `rinetd` 都是 `inetd("the Internet daemon")` 的替代品。不过 `inetd` 最早跟随 `4.3BSD`（1986年）推出，在如今的 **主流 Linux 发行版** 中都已经被弃用了；而 `rinetd` 虽然在各大包管理器中均能找到，但它的作者也说了，自从上个世纪九十年代之后就没怎么动过了；对于 `xinetd`，虽然最后一个版本 `2.3.15` 发布于 **2012年5月9日**，但[Github主页](https://github.com/xinetd-org/xinetd)上最近一次更新还在2016年。

`xinetd` 在 `2.1.8.8pre*` 版本（早于2000年6月4号），就增加了 `IPv6` 支持，太超前太牛逼了，详见[互联网档案馆 2000-08-16 快照](https://web.archive.org/web/20000816232758/http://www.xinetd.org/) 和 [互联网档案馆 2013-06-07 快照](https://web.archive.org/web/20130629172033/http://www.xinetd.org/#changes)（因官网 `http://www.xinetd.org` 早已不可访问，所以只能在 `互联网档案馆` 里找到 `2000-08-16 ~ 2013-06-09` 之间的网页快照）

### 安装
- `Manjaro Linux` / `Arch Linux` 
```
$ sudo pacman -S xinetd
```
- `Debian` / `Ubuntu`
```
$ sudo apt update
$ sudo apt install xinetd
```
- `RHEL` / `CentOS`
```
$ sudo yum update
$ sudo yum install xinetd
```
- `FreeBSD`
```
$ sudo pkg update
$ sudo pkg install xinetd
```

### 配置
配置文件默认位于 `/etc/xinetd.d` 目录下，且安装完会默认创建一些常用的配置（默认均为禁用状态：`disable = yes`）
```
$ ls -lh /etc/xinetd.d
-rw-r--r-- 1 root root 306 Nov 14  2019 rlogin
-rw-r--r-- 1 root root 303 Nov 14  2019 rsh
-rw-r--r-- 1 root root 315 Aug  8 03:49 rsync
-rw-r--r-- 1 root root 205 Sep  5 17:34 sane
-rw-r--r-- 1 root root 253 Nov 13  2019 servers
-rw-r--r-- 1 root root 254 Nov 13  2019 services
-rw-r--r-- 1 root root 157 Nov 14  2019 talk
-rw-r--r-- 1 root root 160 Nov 14  2019 telnet
-rw-r--r-- 1 root root 158 Sep  6 23:32 tftp
```

针对我的诉求，创建配置文件 
```
# cat >> /etc/xinetd.d/win10-gk41 <<EOF
service services
{
        flags           = IPv6
        disable         = no
        type            = UNLISTED
        socket_type     = stream
        protocol        = tcp
        user            = nobody
        wait            = no
        redirect        = 192.168.0.41 3389
        port            = 3389
}
EOF
```
> 该配置表示，监听本地网卡 `IPv6地址` 的 `3389` 端口，然后转发到 `192.168.0.41:3389` 

然后重启 `xinetd` 生效，在 `微软远程桌面` 里通过访问 `manjaro-gk41.example.com` 即可访问到 `gk41` 上运行的 `Windows 10`。

---

最后，如果该文对读者有些许帮助，考虑下给点捐助鼓励一下呗😊
![](https://image.blog.chaosjohn.com/donate-me.png)