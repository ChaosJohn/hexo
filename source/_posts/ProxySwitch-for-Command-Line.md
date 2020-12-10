---
title: 命令行下快捷开关代理
date: 2020-12-09 17:36:41
tags: [shell, proxy, zsh]
thumbnail: https://image.blog.chaosjohn.com/ProxySwitch-for-Command-Line/banner.jpg
---

[欢迎转载，但请在开头或结尾注明原文出处【blog.chaosjohn.com】](https://blog.chaosjohn.com/ProxySwitch-for-Command-Line.html)

## 前言
笔者经常会遇到在命令行下不得不走代理的情况：
- `Homebrew` & `Cask`（特别是后者，不走代理的话速度简直感人）
- 从 `Github` 执行各种操作（`clone` / `pull` / `push` / `fetch`）

## 旧方案
笔者在本地的 `1080` 端口搭建了 `socks5` 代理
开启代理：
```
export all_proxy=socks5://localhost:1080 && export ALL_PROXY=$all_proxy
```

有时候要用到局域网的另外一台电脑上搭建的代理，还得将 `localhost` 改为那台电脑的IP。

虽然 `zsh` 可以通过键入前几个字母比如 `export all_` 然后通过 `方向上健` 快速定位到之间的历史，然后向左移动光标，删去 `localhost`，重新输入新的 `IP`，但是笔者还是嫌麻烦，所以动了造轮子的心思。

## 新方案
因为笔者采用的是 `zsh`，所以在 `$HOME/.zshrc` 内添加以下函数
```
function socks5on() {
  _PORT="1080"
  _HOST="127.0.0.1"

  if [ $# = 0 ]; then
  elif [ $# = 1 ]; then
    _PORT=$1
  elif [ $# = 2 ]; then
    _HOST=$1
    _PORT=$2
  fi
  export all_proxy=socks5://${_HOST}:${_PORT}
  export ALL_PROXY=$all_proxy
  echo $ALL_PROXY
}

function socks5off() {
  unset all_proxy
  unset ALL_PROXY
}
```

函数 `socks5on` 的用法：
- `$ socks5on` 设置代理为 `socks5://127.0.0.1:1080`
- `$ socks5on 1081` 设置代理为 `socks5://127.0.0.1:1081`
- `$ socks5on 192.168.0.253 1080` 设置代理为 `socks5://192.168.0.253:1080`

函数 `socks5off` 的用法：
- `$ socks5off` 取消代理设置

> 注意：
> 1. `socks5on` 函数里，笔者变量取名 `_HOST` 而不是 `HOST`，是因为 `$HOST` 默认存放系统主机名称。
> 
> 2. `all_proxy` 和 `ALL_PROXY` 同时都设置，是因为，有的程序读取 `$all_proxy`，有的程序读取 `$ALL_PROXY`。

读者可以拿去自行更改食用哈。 

---

最后，如果该文对读者有些许帮助，考虑下给点捐助鼓励一下呗😊
![](https://image.blog.chaosjohn.com/donate-me.png)