---
title: PHPBrew 使用指南
date: 2020-12-01 10:50:12
tags: [dev, php, package manager]
thumbnail: https://image.blog.chaosjohn.com/Using-phpbrew/banner.png
---

## 前言
咋一看，这个名字取得，肯定是跟 `Homebrew` 学的。既然 `Homebrew` 的定位是 **macOS** 上的 **包管理器**，那 `PHPBrew` 肯定也跟 **包管理器** 沾点边。没错，它可以用来 *构建* 和 *安装* 多个版本的 `php` 到用户主目录(**$HOME**)下。

[项目主页](https://phpbrew.github.io/phpbrew/)

抄一下 `PHPBrew` 的特性:
- 编译配置选项简化为variant，不用再过于担心搜寻路径
- 通过不同的variant(比如PDO, mysql, sqlite, debug等等)进行构建php
- 编译 **apache** 和 **nginx** 模块，并且按不同版本区分
- 构建和安装到用户主目录下，免去root权限
- 版本间易于切换，且与bash/zsh集成
- 自动特性检测
- 易于安装和启用 `php扩展` 到当前环境
- 支持安装多个版本的 php 到系统环境中
- 为 `Homebrew` 和 `MacPorts` 优化路径检测

## 环境需求
- 平台支持
  - Mac OS 10.5+
  - Ubuntu
  - Debian
  - RHEL / CentOS
- 依赖需求
  - PHP5.3+
  - bz2
  - curl
  - gcc, binutils, autoconf, libxml, zlib, readline

各平台上的依赖安装请参考[官方文档](https://github.com/phpbrew/phpbrew/wiki/Requirement)，这里笔者仅附上 `Mac OS + Homebrew` 和 `Ubuntu 20.04`的安装命令
- Mac OS
```
$ xcode-select --install
$ brew install autoconf pkg-config
```
- Ubuntu 20.04
```
$ sudo apt update
$ sudo apt install build-essential libbz2-dev libreadline-dev libsqlite3-dev libssl-dev libxml2-dev php7.4-cli php7.4-bz2 pkg-config
```

## 安装
下载
```
$ curl -L -O https://github.com/phpbrew/phpbrew/releases/latest/download/phpbrew.phar
$ chmod +x phpbrew.phar
```
安装到配置在 `$PATH` 环境变量里的bin目录，比如官方文档里的 `/usr/local/bin`（笔者用的是 `$HOME/bin`）
```
sudo mv phpbrew.phar /usr/local/bin/phpbrew
```

## 设置
初始化shell环境
```
$ phpbrew init
```
`init` 指令将会创建 **$HOME/.phpbrew** 目录

将下面一行加入 `.bashrc` 或者 `.zshrc`
```
[[ -e ~/.phpbrew/bashrc ]] && source ~/.phpbrew/bashrc
```

## 设置搜寻路径前缀
设置默认的搜寻路径前缀，用于搜寻依赖库，可选项有 `macports`, `homebrew`, `debian`, `ubuntu` 或者 自定义路径
- Homebrew用户
```
$ phpbrew lookup-prefix homebrew
```
- MacPorts用户
```
$ phpbrew lookup-prefix macports
```

## 基本用法
列举已知的版本
```
$ phpbrew known
===> Fetching release list...
Downloading https://www.php.net/releases/index.php?json=1&version=8&max=100 via php stream
Downloading https://www.php.net/releases/index.php?json=1&version=7&max=100 via php stream
8.0: 8.0.0 ...
7.4: 7.4.13, 7.4.12, 7.4.11, 7.4.10, 7.4.9, 7.4.8, 7.4.7, 7.4.6 ...
7.3: 7.3.25, 7.3.24, 7.3.23, 7.3.22, 7.3.21, 7.3.20, 7.3.19, 7.3.18 ...
7.2: 7.2.34, 7.2.33, 7.2.32, 7.2.31, 7.2.30, 7.2.29, 7.2.28, 7.2.27 ...
7.1: 7.1.33, 7.1.32, 7.1.31, 7.1.30, 7.1.29, 7.1.28, 7.1.27, 7.1.26 ...
7.0: 7.0.33, 7.0.32, 7.0.31, 7.0.30, 7.0.29, 7.0.28, 7.0.27 ...
```
想看到更多小版本
```
$ phpbrew known --more
Read local release list (last update: 2020-12-01 03:46:52 UTC).
You can run `phpbrew update` or `phpbrew known --update` to get a newer release list.
8.0: 8.0.0
7.4: 7.4.13, 7.4.12, 7.4.11, 7.4.10, 7.4.9, 7.4.8, 7.4.7, 7.4.6, 7.4.5, 7.4.4, 7.4.3,
     7.4.2, 7.4.1, 7.4.0
7.3: 7.3.25, 7.3.24, 7.3.23, 7.3.22, 7.3.21, 7.3.20, 7.3.19, 7.3.18, 7.3.17, 7.3.16,
     7.3.15, 7.3.14, 7.3.13, 7.3.12, 7.3.11, 7.3.10, 7.3.9, 7.3.8, 7.3.7, 7.3.6,
     7.3.5, 7.3.4, 7.3.3, 7.3.2, 7.3.1, 7.3.0
7.2: 7.2.34, 7.2.33, 7.2.32, 7.2.31, 7.2.30, 7.2.29, 7.2.28, 7.2.27, 7.2.26, 7.2.25,
     7.2.24, 7.2.23, 7.2.22, 7.2.21, 7.2.20, 7.2.19, 7.2.18, 7.2.17, 7.2.16, 7.2.15,
     7.2.14, 7.2.13, 7.2.12, 7.2.11, 7.2.10, 7.2.9, 7.2.8, 7.2.7, 7.2.6, 7.2.5,
     7.2.4, 7.2.3, 7.2.2
7.1: 7.1.33, 7.1.32, 7.1.31, 7.1.30, 7.1.29, 7.1.28, 7.1.27, 7.1.26, 7.1.25, 7.1.24,
     7.1.23, 7.1.22, 7.1.21, 7.1.20, 7.1.19, 7.1.18, 7.1.17, 7.1.16, 7.1.15, 7.1.14
7.0: 7.0.33, 7.0.32, 7.0.31, 7.0.30, 7.0.29, 7.0.28, 7.0.27
```
更新最新版本发行列表
```
$ phpbrew update
===> Fetching release list...
Downloading https://www.php.net/releases/index.php?json=1&version=8&max=100 via php stream
Downloading https://www.php.net/releases/index.php?json=1&version=7&max=100 via php stream
8.0: 1 releases
7.4: 14 releases
7.3: 26 releases
7.2: 33 releases
7.1: 20 releases
7.0: 7 releases
===> Done
```

获取小于7.0的老版本（官方文档已滞后，还仅仅只把 **小于5.4** 作为老版本）
> 注意：不保证能成功构建这些不被PHPBrew官方支持的版本
```
$ phpbrew update --old
```

列举小于7.0的老版本
```
$ phpbrew known --old
===> Fetching release list...
Downloading https://www.php.net/releases/index.php?json=1&version=8&max=100 via curl extension
Downloading https://www.php.net/releases/index.php?json=1&version=7&max=100 via curl extension
Downloading https://www.php.net/releases/index.php?json=1&version=5&max=1000 via curl extension
8.0: 8.0.0 ...
7.4: 7.4.13, 7.4.12, 7.4.11, 7.4.10, 7.4.9, 7.4.8, 7.4.7, 7.4.6 ...
7.3: 7.3.25, 7.3.24, 7.3.23, 7.3.22, 7.3.21, 7.3.20, 7.3.19, 7.3.18 ...
7.2: 7.2.34, 7.2.33, 7.2.32, 7.2.31, 7.2.30, 7.2.29, 7.2.28, 7.2.27 ...
7.1: 7.1.33, 7.1.32, 7.1.31, 7.1.30, 7.1.29, 7.1.28, 7.1.27, 7.1.26 ...
7.0: 7.0.33, 7.0.32, 7.0.31, 7.0.30, 7.0.29, 7.0.28, 7.0.27 ...
5.6: 5.6.40, 5.6.39, 5.6.38, 5.6.37, 5.6.36, 5.6.35, 5.6.34, 5.6.33 ...
5.5: 5.5.38, 5.5.37, 5.5.36, 5.5.35, 5.5.34, 5.5.33, 5.5.32, 5.5.31 ...
5.4: 5.4.45, 5.4.44, 5.4.43, 5.4.42, 5.4.41, 5.4.40, 5.4.39, 5.4.38 ...
5.3: 5.3.29, 5.3.28, 5.3.27, 5.3.26, 5.3.25, 5.3.24, 5.3.23, 5.3.22 ...
5.2: 5.2.17, 5.2.16, 5.2.15, 5.2.14, 5.2.13, 5.2.12, 5.2.11, 5.2.10 ...
5.1: 5.1.6, 5.1.5, 5.1.4, 5.1.3, 5.1.2, 5.1.1, 5.1.0 ...
5.0: 5.0.5, 5.0.4, 5.0.3, 5.0.2, 5.0.1, 5.0.0 ...
PHPBrew needs PHP 5.3 or above to run. build/switch to versions below 5.3 at your own risk.
```

## 开始构建你自己的PHP
用默认variant简单构建并安装PHP：
```
$ phpbrew install 7.4.13 +default
```
这里我们推荐使用 **default** 设置，它涵盖了大部分广泛使用的variant；如果你需要一个 *最小化* 安装，那就移除 **default**，替换成别的variant（经笔者摸索，*最小化* 为 **+opcache -xml**）。

通过传递参数 **-j** 或者 **--jobs** 进行并行编译：
```
$ phpbrew install -j $(nproc) 7.4.13 +default
```

包含测试：
```
$ phpbrew install --test 7.4.13 +default
```

包含调试信息：
```
$ phpbrew -d install --test 7.4.13 +default
```

安装小于5.3的老版本（笔者实测5.0版本的链接均已不可访问）
```
$ phpbrew install --old 5.2.17
```

安装 `next` 不稳定版：
```
$ phpbrew install next
```

从GitHub仓库的某tag安装
```
$ phpbrew install github:php/php-src@PHP-7.0
```

用 `as` 给构建安装的PHP取别名，以笔者在前面描述的 *最小化* 安装举例
```
$ phpbrew install 7.4.13 +opcache -xml as 7.4.13-minimal
```

## 清理构建目录
```
$ phpbrew clean 7.4.13-minimal
===> Running make clean: /usr/bin/make -C '/Users/chaos/.phpbrew/build/7.4.13-minimal' --quiet 'clean'
Distribution is cleaned up. Woof!
```

## Variants
PHPBrew 帮你安排构建参数，你可以简单地指定variant，PHPBrew会自动检测 `include` 路径并且构建编译选项。

PHPBrew 提供了默认的和虚拟的variant。默认的包含了大部分广泛使用的variants；虚拟的则定义了一组variant，即使用一个虚拟variant可以同时选用多个variant。

查看PHPBrew提供了哪些variant：
```
$ phpbrew variants
Variants:
  all, apxs2, bcmath, bz2, calendar, cgi, cli, ctype, curl, dba, debug, dom,
  dtrace, editline, embed, exif, fileinfo, filter, fpm, ftp, gcov, gd,
  gettext, gmp, hash, iconv, imap, inifile, inline, intl, ipc, ipv6, json,
  kerberos, ldap, libgcc, mbregex, mbstring, mcrypt, mhash, mysql, opcache,
  openssl, pcntl, pcre, pdo, pear, pgsql, phar, phpdbg, posix, readline,
  session, soap, sockets, sodium, sqlite, static, tidy, tokenizer, wddx,
  xml, xmlrpc, zip, zlib, zts


Virtual variants:
  dbs: sqlite, mysql, pgsql, pdo
  mb: mbstring, mbregex
  neutral:
  small: bz2, cli, dom, filter, ipc, json, mbregex, mbstring, pcre, phar,
  posix, readline, xml, curl, openssl
  default: bcmath, bz2, calendar, cli, ctype, dom, fileinfo, filter, ipc,
  json, mbregex, mbstring, mhash, pcntl, pcre, pdo, pear, phar, posix,
  readline, sockets, tokenizer, xml, curl, openssl, zip
  everything: dba, ipv6, dom, calendar, wddx, static, inifile, inline, cli,
  ftp, filter, gcov, zts, json, hash, exif, mbstring, mbregex, libgcc,
  pdo, posix, embed, sockets, debug, phpdbg, zip, bcmath, fileinfo, ctype,
  cgi, soap, pcntl, phar, session, tokenizer, opcache, imap, ldap, tidy,
  kerberos, xmlrpc, fpm, dtrace, pcre, mhash, mcrypt, zlib, curl, readline,
  editline, gd, intl, sodium, openssl, mysql, sqlite, pgsql, xml, gettext,
  iconv, bz2, ipc, gmp, pear


Using variants to build PHP:

  phpbrew install php-5.3.10 +default
  phpbrew install php-5.3.10 +mysql +pdo
  phpbrew install php-5.3.10 +mysql +pdo +apxs2
  phpbrew install php-5.3.10 +mysql +pdo +apxs2=/usr/bin/apxs2
```
可以看到虚拟variant有：dbs, mb, neutral, small, default, everything

（表明上来看 **neutral** 是最精简的，它不添加任何编译选项，但实际安装过程中还是会启用 **xml** 和 **opcache**，而这二者中只有后者无任何第三方依赖。所以真正意义上的 **最小化** 为 **+opcache -xml** 或者 **+neutral -xml**）

- 需要启用一个variant，直接在variant前加一个 `+` 号
- 需要禁用一个variant，直接在variant前加一个 `-` 号
- `+` 和 `-` 号可以多个随意组合

指定依赖路径，比如构建pgsql扩展时
```
$ phpbrew install 7.4.13 +pdo +pgsql=/opt/local/lib/postgresql91/bin
```
在 `/opt/local/lib/postgresql91/bin` 路径下的 `pg_config` 是构建时需要的

## 传递额外的configure选项
```
$ phpbrew install 7.4.13 +mysql +sqlite -- \
    --enable-ftp --apxs2=/opt/local/apache2/bin/apxs
```

## 使用与切换
使用（临时切换版本）
```
$ phpbrew use 7.4.13
$ phpbrew use 7.4.13-minimal
```

切换默认版本
```
$ phpbrew switch 7.4.13-minimal
```

从PHPBrew的版本切换到系统安装的版本
```
$ phpbrew off
```

## 列举安装的所有php版本
```
$ phpbrew list
```

## 扩展安装器
PHPBrew 还能方便的帮助你安装 **PHP扩展**（无论是PHP源码内自带的亦或是来自于 `PECL` 的）

如果是PHP源码内自带的，PHPBrew会自动切换到源码目录进而安装扩展；否则会去 `PECL` [http://pecl.php.net](http://pecl.php.net)去获取扩展源码包

PHPBrew同时会为安装上的扩展创建配置，省去你手写启用扩展的配置文件。配置文件夹位于: `$HOME/.phpbrew/php/{php-version}/var/db`

### 安装扩展 - 最简单的方式
在你安装任何扩展之前，你应该切换到需要安装扩展的php版本：
```
$ phpbrew use 7.4.13-minimal
```

然后运行 `ext install` 来安装自己需要的扩展
```
$ phpbrew ext install redis
$ phpbrew ext install xdebug
$ phpbrew ext install acpu
$ phpbrew ext install memcache
```

### 安装扩展 - 指定稳定性版本
- 指定 **稳定性标签**，可选项有 `stable` / `latest` / `beta`，不指定即默认为 **stable**，例如：`$ phpbrew ext install xdebug latest`
- 指定 **扩展版本号**，例如：`$ phpbrew ext install xdebug 3.0.0`

### 查看扩展的配置选项
要查看构建扩展是否有额外的配置选项，使用 `ext show`
```
$ phpbrew ext show redis
                Name: redis
    Source Directory: /Users/chaos/.phpbrew/build/7.4.13/ext/redis
              Config: /Users/chaos/.phpbrew/build/7.4.13/ext/redis/config.m4
            INI File: /Users/chaos/.phpbrew/php/7.4.13/var/db/redis.ini
           Extension: Pecl
                Zend: no
              Loaded: yes

   Configure Options:

        --enable-redis-igbinary[=no]     enable igbinary serializer support?

        --enable-redis-lzf[=no]          enable lzf compression support?

        --enable-redis-zstd[=no]         enable zstd compression support?
```

### 为构建扩展添加额外配置选项
```
$ phpbrew ext install redis -- --enable-redis-lzf=yes
```

### 从Github安装扩展
特殊的前缀 `github:` 则告诉PHPBrew要去 `php-memcached-dev/phpmemcached` 的 `php7` 分支拉取代码进行构建
```
$ phpbrew ext install github:php-memcached-dev/php-memcached php7 -- --disable-memcached-sasl
```

### 指定下载器进行安装扩展
目前，PHPBrew有4个下载器实现：
- `php_curl` 内置的 *php curl扩展*，为默认下载器
- `php_stream` 内置的 *php steam封装*
- `curl`
- `wget`

指定 `curl` 作为下载器：
```
$ phpbrew ext install --downloader curl redis
```

如果选用的下载器支持 `User-Agent` 和 `Proxy` 配置：
```
$ phpbrew ext install --downloader php_curl --http-proxy=... --http-proxy-auth=... apcu
```

### 启用扩展
笔者亲测，通过 `ext install` 安装的扩展，安装完即为启用状态。

另外也可以通过 `PECL` 安装扩展并且手动启用：
```
$ pecl install mongo
$ phpbrew ext enable mongo
```
`ext enable` 指令帮你创建配置文件 **{current php base}/var/db/{extension name}.ini** 用以启用扩展

### 为当前php版本配置 `php.ini`
```
$ export EDITOR=vim # 该行为可选项，指定你常用的编辑器
$ phpbrew config
```

## 更新 PHPBrew
运行 `self-update` 指令将会从Github的master分支下载最新版本的二进制文件替换自身（即本文开头的存放位置`/usr/local/bin/phpbrew`）


## 相关的目录
已安装的php位于 `$HOME/.phpbrew/php`，以 **7.4.13-minimal** 举例，为 `$HOME/.phpbrew/php/7.4.13-minimal/bin/php`

配置文件为 `$HOME/.phpbrew/php/7.4.13-minimal/etc/php.ini`

扩展的配置文件位于 `$HOME/.phpbrew/php/7.4.13-minimal/var/db`目录下：
```
$HOME/.phpbrew/php/7.4.13-minimal/var/db/memcache.ini
$HOME/.phpbrew/php/7.4.13-minimal/var/db/xdebug.ini
$HOME/.phpbrew/php/7.4.13-minimal/var/db/redis.ini
```

## ~在各目录之间切换的快捷命令~
~官方文档里这部分涉及的指令，比如 `build-dir` / `dist-dir` / `etc-dir` / `var-dir` 已亲测全部阵亡~

## PHP fpm
PHPBrew同时也为 `php-fpm` 提供了管理指令（前提是在构建的时候得加上 `+fpm` variant）

**启动/停止/重启** php-fpm：
```
$ phpbrew fpm start
$ phpbrew fpm stop 
$ phpbrew fpm restart 
```

显示 php-fpm 模块：
```
$ phpbrew fpm module
[PHP Modules]
bz2
cgi-fcgi
Core
ctype
curl
...

[Zend Modules]
Xdebug
```

测试 php-fpm 配置是否正确：
```
$ phpbrew fpm test
[01-Dec-2020 18:28:04] NOTICE: configuration file /Users/chaos/.phpbrew/php/7.4.13/etc/php-fpm.conf test is successful
```

编辑 php-fpm 配置：
```
$ export EDITOR=vim # 该行为可选项，指定你常用的编辑器
$ phpbrew fpm config
```

> 已安装的 `php-fpm` 位于 `$HOME/.phpbrew/php/{php-version}/sbin` 下
> 
> 相应的 `php-fpm.conf` 位于 `$HOME/.phpbrew/php/{php-version}/etc/php-fpm.conf.default`, 你可以将默认配置文件拷贝到需要的位置进行使用，比如
>
>     $ cp -v ~/.phpbrew/php/{php-version}/etc/php-fpm.conf.default ~/.phpbrew/php/{php-version}/etc/php-fpm.conf
>
>     $ php-fpm --php-ini ~/.phpbrew/php/{php-version}/etc/php.ini --fpm-config ~/.phpbrew/php/{php-version}/etc/php-fpm.conf

## ~安装扩展应用~
~通过 `app get` 指令来安装例如 `composer` 和 `phpunit` 这样的 app，笔者亲测该指令也已阵亡~

## 结尾
以上便是笔者结合 PHPBrew 官方文档给大家梳理的使用指南，写该文时每条指令笔者都亲自测试过，所以发现了官方文档里有很多地方都 **已过时** 或 **已更新**。对于前者，笔者用横线给划掉了；对于后者，笔者直接进行了修正。

---

最后，如果该文对读者有些许帮助，考虑下给点捐助鼓励一下呗😊
![](https://image.blog.chaosjohn.com/donate-me.png)

