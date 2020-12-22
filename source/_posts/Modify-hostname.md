---
title: 在 Linux 和 Mac 上更改 hostname
date: 2020-12-21 21:26:22
tags: [shell, Linux, OS X & macOS, hostname]
thumbnail: https://image.blog.chaosjohn.com/Modify-hostname/banner.png
---

[欢迎转载，但请在开头或结尾注明原文出处【blog.chaosjohn.com】](https://blog.chaosjohn.com/Modify-hostname.html)

## 前言
看过我文章的朋友都可能发现，从我的终端截图可以看出，我使用的是 `oh-my-zsh`，并且对 `RPROMPT` 进行了自定义
![oh-my-zsh 自定义 PROMPT](https://image.blog.chaosjohn.com/Modify-hostname/oh-my-zsh-prompt.png)

笔者设置的是: 日期 + 时间 + 主机名(hostname)

因为笔者有很多电脑 + 虚拟机 + 云主机，所以在终端的命令提示符显示 `hostname` 对于笔者就格外重要，可以方便笔者一目了然的知道当前终端连接的是哪台机子。

当然，前提是，每台机子都要预先设定 `hostname`

## 设定方法
### 通用
***nix** 系统（Linux & macOS & FreeBSD & ...）通用的设定 `hostname` 的方法为
```
# hostname <NEW HOSTNAME>
```

### Linux
如果安装了 `systemd` 的话，可以借助 `hostnamectl`

我们来看一下它的帮助：
```
$ hostnamectl -h
hostnamectl [OPTIONS...] COMMAND ...

Query or change system hostname.

  -h --help              Show this help
     --version           Show package version
     --no-ask-password   Do not prompt for password
  -H --host=[USER@]HOST  Operate on remote host
  -M --machine=CONTAINER Operate on local container
     --transient         Only set transient hostname
     --static            Only set static hostname
     --pretty            Only set pretty hostname

Commands:
  status                 Show current hostname settings
  set-hostname NAME      Set system hostname
  set-icon-name NAME     Set icon name for host
  set-chassis NAME       Set chassis type for host
  set-deployment NAME    Set deployment environment for host
  set-location NAME      Set location for host
```

所以我们设定 `hostname`，可以用：
```
# hostnamectl set-hostname <NEW HOSTNAME>
```

### macOS
`macOS` 的话，还可以借助于 `scutil`

看一下帮助：
```
$ scutil --help
usage: scutil
	interactive access to the dynamic store.

   or: scutil --prefs [preference-file]
	interactive access to the [raw] stored preferences.

   or: scutil [-W] -r nodename
   or: scutil [-W] -r address
   or: scutil [-W] -r local-address remote-address
	check reachability of node, address, or address pair (-W to "watch").

   or: scutil -w dynamic-store-key [ -t timeout ]
	-w	wait for presense of dynamic store key
	-t	time to wait for key

   or: scutil --get pref
   or: scutil --set pref [newval]
   or: scutil --get filename path key
	pref	display (or set) the specified preference.  Valid preferences
		include:
			ComputerName, LocalHostName, HostName
	newval	New preference value to be set.  If not specified,
		the new value will be read from standard input.

   or: scutil --dns
	show DNS configuration.

   or: scutil --proxy
	show "proxy" configuration.

   or: scutil --nwi
	show network information

   or: scutil --nc
	show VPN network configuration information. Use --nc help for full command list

   or: scutil --allow-new-interfaces [off|on]
	manage new interface creation with screen locked.

   or: scutil --error err#
	display a descriptive message for the given error code
```

我们可以看到，`scutil` 可以设置 `ComputerName` / `LocalHostName` / `HostName`。那三者有什么区别呢？

- `ComputerName`：电脑名称，如果用过 `Time Machine` 的话，可以在其备份文件夹内看到以 `ComputerName.backupbundle` 命名的文件，其存的就是对应电脑的历史备份 ![Time Machine 备份](https://image.blog.chaosjohn.com/Modify-hostname/time-machine-backup-as-computer-name.png)
- `LocalHostName`：局域网(`Bonjour`)内对外宣告的主机名，例如 `machine007.local`，则局域网内其他机子，可以通过 `machine007.local` 直接解析出局域网IP，进而直接访问到本机
- `HostName`：通常是 `LocalHostName` 去除尾部的 `.local`，也就是我们要设定的 `hostname`

![“系统设置->共享”](https://image.blog.chaosjohn.com/Modify-hostname/system-preferences-sharing.png)

所以我们设定 `hostname`，可以用：
```
# scutil --set HostName <NEW HOSTNAME>
```

---

最后，如果该文对读者有些许帮助，考虑下给点捐助鼓励一下呗😊
![](https://image.blog.chaosjohn.com/donate-me.png)