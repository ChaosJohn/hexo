---
title: 解决 Composer 报错 InvalidArgumentException
date: 2020-12-03 17:37:32
tags: [php, fix, composer]
thumbnail: https://image.blog.chaosjohn.com/Fix-InvalidArgumentException-for-Composer/logo-composer-transparent.png
---

[欢迎转载，但请在开头或结尾注明原文出处【blog.chaosjohn.com】](https://blog.chaosjohn.com/Fix-InvalidArgumentException-for-Composer.html)

[composer](https://docs.phpcomposer.com/) 在 **2020-11-24** 发布了全新的 `2.0` 版本。[参考官方博客](https://blog.packagist.com/composer-2-0-is-now-available/)

鉴于博客里描述新版在很多方面都做出了优化，特别是性能方面，提速了很多，于是笔者迫不及待地更新了。

![测试的是无缓存下的耗时（"初始化update" + "install"），表现出了60%的提速](https://image.blog.chaosjohn.com/Fix-InvalidArgumentException-for-Composer/composer2-improvements.png)

但是在写上一篇文章 [php 调试指南（Xdebug版）](https://blog.chaosjohn.com/Debug-php.html)的篇头处，执行 `composer require mikecao/flight` 却发生了异常
```
  [RuntimeException]
  No composer.json present in the current directory, this may be the cause of the following exception.



  [InvalidArgumentException]
  Could not find package mikecao/flight.

  Did you mean one of these?
      mikecao/flight
      geogkary/breeze
```

分析报错信息，提示找不到 `mikecao/flight`，又问我是不是想找 `mikecao/flight`。

这。。。有毒吧

会不会是 `Composer 2.0` 的 **bug**？不科学呀，不至于有这么大的bug还发布了出来。

会不会是我的配置有问题？查看一下 **配置文件**
```
$ cat ~/.composer/config.json
{
    "config": {},
    "repositories": {
        "packagist": {
            "type": "composer",
            "url": "https://packagist.phpcomposer.com"
        }
    }
}
```

可以看到除了 `镜像源` 以外并无其他的配置。

问题会不会出在 `镜像源` 上呢，反正排除法就那么多选项
1. 删除 `镜像源`
2. 删除 `composer` 二进制文件以及程序目录 `$HOME/.composer` 后重装

将 `config.json` 的 `packagist` 块删除后（亦可执行 `composer config -g --unset repos.packagist`），再次执行 `require`
```
$ composer require mikecao/flight
Using version ^1.3 for mikecao/flight
./composer.json has been updated
Running composer update mikecao/flight
Loading composer repositories with package information
Updating dependencies
Lock file operations: 1 install, 0 updates, 0 removals
  - Locking mikecao/flight (v1.3.8)
Writing lock file
Installing dependencies from lock file (including require-dev)
Package operations: 1 install, 0 updates, 0 removals
  - Installing mikecao/flight (v1.3.8): Extracting archive
Generating autoload files
```

Bingo! 异常报错消失了

可是该镜像源 [Packagist / Composer 中国全量镜像](https://pkg.phpcomposer.com/) 笔者已经使用了好多年了，怎么突然就挂了呢？网上也搜不到它的 **停服** 消息。

考虑到国内的网络环境，得重新找一个镜像源。

经过一番网罗，笔者最终敲定选用 [阿里云 Composer 全量镜像](https://developer.aliyun.com/composer)，配置命令：
```
$ composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
```

其他的推荐镜像源：
- 华为云 [https://mirrors.huaweicloud.com/repository/php/](https://mirrors.huaweicloud.com/repository/php/)

次选镜像源：
- 腾讯云 [https://mirrors.cloud.tencent.com/composer/](https://mirrors.cloud.tencent.com/composer/)
- 上海交通大学 [https://packagist.mirrors.sjtug.sjtu.edu.cn](https://packagist.mirrors.sjtug.sjtu.edu.cn)

列为次选是因为还未支持 **Composer2**，运行时会有 **黄色警告**
```
Composer 2 repository support for https://mirrors.xxx.com/composer has been disabled due to what seems like a misconfiguration. If this is a packagist.org mirror we recommend removing it as Composer 2 handles network operations much faster and should work fine without.
```

---

最后，如果该文对读者有些许帮助，考虑下给点捐助鼓励一下呗😊
![](https://image.blog.chaosjohn.com/donate-me.png)
