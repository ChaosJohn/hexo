---
title: php 调试指南
date: 2020-12-01 23:33:41
tags: [debug, php, xdebug]
thumbnail: https://image.blog.chaosjohn.com/Debug-php/banner.png
---

## 创建一个精简项目（命令行）
创建项目，并且用 [composer](https://docs.phpcomposer.com/) 安装一个笔者比较喜欢的 **微框架**，作为示例
```
$ mkdir debug
$ cd debug
$ composer require mikecao/flight
```

编写主程序 **index.php**
```
cat >> index.php <<EOF
<?php
require 'vendor/autoload.php';

Flight::route('/', function(){
    echo 'hello world!';
});

Flight::start();
EOF
```

用 php 内置服务器运行项目
```
$ php -S localhost:8080
[Tue Dec  1 23:31:12 2020] PHP 7.4.13 Development Server (http://localhost:8080) started
[Tue Dec  1 23:31:35 2020] [::1]:57780 Accepted
[Tue Dec  1 23:31:35 2020] [::1]:57780 [200]: GET /
[Tue Dec  1 23:31:35 2020] [::1]:57780 Closing
```
![创建并且运行项目][img01]
上面输出的后三行，是在本机另一终端访问 `http://localhost:8080` 时打印的日志
```
$ curl -L localhost:8080
hello world!
```
（使用 php 内置服务器运行项目，只是为了检测项目是否能完好运行；该内置服务器只能用于开发环境，不能用于生产环境）

## 转移开发至 `PhpStorm` 中
**JetBrains** 公司出品的 `PhpStorm` 与其安装在此不做赘述。

在 `PhpStorm` 中打开项目
![PhpStorm 欢迎窗口选择“打开”][img02]

打开 **偏好设置**（mac下快捷键为 `Command+`, Win/Linux下快捷键为 `Ctrl+Alt+s`），左边侧边栏定位到 `Languages & Frameworks` => `PHP`
![打开 PhpStorm 偏好设置][img03]

在这里添加此前于 PHPBrew 中安装的 PHP 版本。参考前文 [PHPBrew 使用指南](https://blog.chaosjohn.com/Using-phpbrew)（注意：请务必安装 `xdebug` 扩展：`$phpbrew ext install xdebug`，否则无法进行调试）
![添加 php 解释器][img04]

将刚刚添加上的 PHP 版本应用到当前项目
![选择 php 解释器][img05]

## 用 `Nginx` 运行项目
### 安装与管理
- 无视系统环境 - 编译安装，不做赘述
- macOS
```
$ brew install nginx
$ brew services start/stop/restart nginx # 启动/停止/重启 nginx
```
- Debian / Ubuntu
```
$ sudo apt update
$ sudo apt install nginx-full
$ sudo systemctl start/stop/restart nginx # 启动/停止/重启 nginx
```
- RHEL / CentOS
```
$ sudo yum update
$ sudo apt install nginx
$ sudo systemctl start/stop/restart nginx # 启动/停止/重启 nginx
```

### 配置（本文以 **macOS** 下 **Homebrew** 版 **Nginx** 为例）
启动 `php-fpm`（用 PHPBrew 安装 PHP 时需要包含 `+fpm` variant）
```
$ phpbrew fpm start
Starting php-fpm...
```

在项目根目录下创建 `nginx.conf` 作为 **Nginx** 的配置文件
```
server {
    server_name localhost;
    listen 8000;

    root /Users/chaos/Work/php/demos/debug;

    location / {
        try_files $uri $uri/ /index.php;
        fastcgi_pass unix:/Users/chaos/.phpbrew/php/7.4.13/var/run/php-fpm.sock;
        fastcgi_index index.php;
        include fastcgi.conf;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```

将该配置文件 **软链接** 到 Nginx 配置目录下 `/usr/local/etc/nginx/servers/`
```
$ ln -sf /Users/chaos/Work/php/demos/debug/nginx.conf /usr/local/etc/nginx/servers/php-debug.conf
```

检查 `nginx.conf` 是否有错误
```
$ nginx -t
nginx: the configuration file /usr/local/etc/nginx/nginx.conf syntax is ok
nginx: configuration file /usr/local/etc/nginx/nginx.conf test is successful
```

启动 `Nginx`
```
$ brew services start nginx
==> Successfully started `nginx` (label: homebrew.mxcl.nginx)
```

![Nginx 配置][img06]

终端下 curl 访问 Nginx部署的服务，验证项目成功跑了起来
```
$ curl -L localhost:8000
hello world!
```

## `Xdebug` 调试
### 大部分关于 `Xdebug` 的配置，在前面部分已经配置完毕，我们来验证一下是否配置成功
**PhpStorm** 顶部菜单栏点击 `Run`，在下拉菜单中选中 `Web Server Debug Validation`，弹出窗口中点击底部的 `Validate` 按钮。
- 如果显示成功，则调试环境已经搭建完毕
- 如果显示出错，会给出排错信息
![出错，显示排错信息][img07]

发现三处警告，按照排错信息排查，发现问题出在了 `Xdebug 3.0.0` 版本[变动了非常多的地方](https://xdebug.org/docs/upgrade_guide)，导致目前 `PhpStorm` 最新的稳定版 `2020.2.4` 对此兼容性欠缺。不过 `2020.3` 有望解决。

所以笔者将 `Xdebug` 降级到了 `2.X` 版本：`phpbrew ext install xdebug 2.9.8`，再次点击 `Validate` 按钮，怎么还是一样的报错？

这里我们查看 `php-fpm` 的信息
```
$ phpbrew fpm info
...
xdebug support => enabled
Version => 3.0.0
...
```

所以需要重启一下 `php-fpm`，让其加载新的 `Xdebug` 模块
```
$ phpbrew fpm restart
Stopping php-fpm...
Starting php-fpm...
$ phpbrew fpm info
...
xdebug support => enabled
Version => 2.9.8
...
```

再次点击 `Validate` 按钮
![2.9.8 依旧报错][img08]

依旧报错，不过这里就清晰很多了，照着提示，`phpbrew config` 直接调用编辑器打开 `php.ini` 文件，在尾部添加：
```
[xdebug]
xdebug.remote_enable=1
```

待 `phpbrew fpm restart` 后再次点击 `Validate` 按钮，成功！
![Validate 成功][img09]

这就可以愉快的调试部署在 Nginx 里的项目啦，这里我们点击工具栏的 `监听按钮`，开始监听连接请求
![监听请求][img10]

### 浏览器作为请求客户端进行调试
为浏览器安装插件
- [Xdebug Helper for Firefox](https://addons.mozilla.org/en-GB/firefox/addon/xdebug-helper-for-firefox/) [源码](https://github.com/BrianGilbert/xdebug-helper-for-firefox)
- [Xdebug Helper for Chrome](https://chrome.google.com/extensions/detail/eadndfjplgieldjbigjakmdgkmoaaaoc) [源码](https://github.com/mac-cain13/xdebug-helper-for-chrome)

这里笔者以 **Chrome** 浏览器为例，安装完后，右键 `Xdebug Helper` 图标点击 `Options` 打开选项页，选择 `IDE key` 为 `PhpStorm`
![Chrome 下 Xdebug Helper 配置][img11]

新开浏览器窗口，地址栏输入`localhost:8000`访问服务，页面会显示 **hello world!**。单击 `Xdebug Helper` 图标，点击 `Debug`，开启调试模式，至此，图标将会变成绿色（或者使用 `Alt+Shift+X` 快捷键快速开关 `Debug` 模式）
![切换 Xdebug Helper 到 Debug 模式][img13]

准备开始调试：
1. 确保 `PhpStorm` 的 `监听` 按钮已激活
2. 在 `PhpStorm` 中给需要调试的代码行打上断点
3. 确保浏览器的 `Xdebug Helper`处于 `Debug` 模式，图标变为绿色

![调试准备][img14]

刷新浏览器页面，`PhpStorm` 监听到请求传入，同时浏览器请求处于 `Pending` 状态，挂起等待请求响应
![开始调试][img15]

在弹窗中选择完正确的 php 文件后，`PhpStorm` 就进入了调试模式，成功地停在了断点处，在这里可以查看当前的各项 **变量** 以及单步运行
![PhpStorm断点调试][img16]

点击调试窗口左侧的 `Resume Program` 按钮，程序从断点处恢复运行，至此，浏览器侧的请求收到了等待已久的响应，在窗口渲染出 *hello world!* 字样

### `命令行` 或其他 `请求客户端` 发起调试
在实际的纯接口开发场景中，除了类似 `localhost:8000` 的简单请求，还有诸多带有复杂参数的请求，以及各种 `http method` 的请求（`GET` / `POST` / `PUT` / `HEAD` / `DELETE` / `OPTIONS`），仅仅通过浏览器地址栏进行发起调试并不能胜任。

开发人品平时发起请求测试，一般都选用 `命令行工具`（例如 `curl` / `httpie`）或类似 `Postman` 这样的专业请求调试客户端。

通过查阅[Xdebug 官方文档](https://xdebug.org/docs/step_debug)，得知，浏览器的插件开启 `Debug` 模式，仅仅是给当前session增加了一个 `Cookie`: `XDEBUG_SESSION=PHPSTORM`

那就简单了，以 `curl` 举例的三种方式：
- `$ curl -H "Cookie: XDEBUG_SESSION=PHPSTORM" localhost:8000`
- `$ curl --cookie XDEBUG_SESSION=PHPSTORM localhost:8000`
- `$ curl -b XDEBUG_SESSION=PHPSTORM localhost:8000`

`Postman` 也类似，不再赘述

至此，在 `PhpStorm` 调试部署在 `Nginx` 中的项目，杀青！

## 提出疑问
还记得前面有一张关于配置 `php解释器` 图么（重新搬运放在本段下方），里面明明配置了 `Xdebug` 的参数，而 `Web Server Debug Validation` 还是提示要去 `php.ini` 中添加 `xdebug.remote_enable=1`
![配置 php 解释器][img04]

`JetBrains` 肯定不会那么无聊，放置一个无用的配置项在这里。那这个配置项究竟对哪块地方生效呢？

`PhpStorm` 工具栏点击 `Add Configuration`，点击 `+` 号，选择 `PHP Build-in Web Server`，端口改为 **8088** (或其他未被占用端口)
![PhpStorm 添加配置][img17] ![PhpStorm 完善配置][img18]

选择刚刚新建的 **运行配置** `php build-in server`，点击右侧 *绿色三角按钮*，运行项目（同时确保 `监听按钮` 也已激活）
![PhpStorm 运行配置][img19]

再次在命令行下执行：
```
$ curl -H "Cookie: XDEBUG_SESSION=PHPSTORM" localhost:8000
```

Bingo!!! `PhpStorm` 成功进入了调试状态并且停在了断点处！

## 提出疑问 2
上文描述的都是调试本地环境，即服务部署在本地 `Nginx`，通过本地的 `php-fpm` 调用 `php 解释器` 进行请求分发处理，“加载”的是本地的 `Xdebug`（调试端口为默认的 **9000**），而同时 `PhpStorm` 也监听本地的 `Xdebug` 的 **9000** 端口进行拦截调试。

可往往更多的情况是，服务并不部署在本地，而在 **远端服务器**，这个时候得如何调试呢？

其实理解了 `Xdebug` 的工作原理，结合[PhpStorm的官方文档](https://www.jetbrains.com/help/phpstorm/run-debug-configuration-php-remote-debug.html)，我们接下来在 **另一台主机(`192.168.1.101`)** 上调试 **本机(`192.168.1.100`)** 的服务，来模拟现实开发中的 `远程调试`。

**另一台主机(`192.168.1.101`)** 的 `PhpStorm` 工具栏点击 `Add Configuration`，点击 `+` 号，选择 `PHP Remote Debug`，添加一个 `Server`（`Host` 填写 `192.168.1.100`，`Port` 填写 `8000`，`Debugger` 选择 `Xdebug`），完成后再点击工具栏上的 `监听按钮` 开启监听，并且在代码窗口设置断点
![PhpStorm 添加配置][img23]

在 **该主机(`192.168.1.101`)** 的终端上执行
```
$ curl -H "Cookie: XDEBUG_SESSION=PHPSTORM" 192.168.1.100:8000
```

即可看到 `PhpStorm` 进入了调试模式并且顺利停在了断点处。


## 发散：那如何在 `VSCode` 中调试呢？
打开 `VSCode`，安装 `PHP Debug` 扩展

打开项目工程，点击左边栏的 `Run`，点击 `Create a launch.json file`，选择 `PHP` 环境
![创建 VSCode 运行配置][img20]

VSCode 将会自动创建两个配置：
- `Listen for XDebug`：这个配置就是类似 `PhpStorm` 中的 `监听按钮`
- `Launch currently open script`：执行按照 `php-cli` 模式运行/调试 **当前窗口的php脚本**

![VSCode 创建的默认运行配置][img21]

选择 `Listen for XDebug` 并点击左侧的 *绿色图标* 开始运行监听，并且打上断点，命令行下执行 `curl --cookie XDEBUG_SESSION=VSCODE localhost:8000`，则能看到**VSCode** 成功进入了调试状态并且停在了断点处。
![VSCode 调试模式][img22]

所以，在 `VSCode` 调试部署在 `Nginx` 中的项目，也顺利杀青！



[img01]: https://image.blog.chaosjohn.com/Debug-php/create-and-run-project.png
[img02]: https://image.blog.chaosjohn.com/Debug-php/open-in-phpstorm.png
[img03]: https://image.blog.chaosjohn.com/Debug-php/navigate-to-preferences.png
[img04]: https://image.blog.chaosjohn.com/Debug-php/add-php-from-phpbrew-with-xdebug.png
[img05]: https://image.blog.chaosjohn.com/Debug-php/apply-php-for-current-project.png
[img06]: https://image.blog.chaosjohn.com/Debug-php/nginx-conf.png
[img07]: https://image.blog.chaosjohn.com/Debug-php/web-server-debug-validation-error.png
[img08]: https://image.blog.chaosjohn.com/Debug-php/web-server-debug-validation-error2.png
[img09]: https://image.blog.chaosjohn.com/Debug-php/web-server-debug-validation-success.png
[img10]: https://image.blog.chaosjohn.com/Debug-php/listening-button.png
[img11]: https://image.blog.chaosjohn.com/Debug-php/chrome-xdebug-helper-options.png
[img12]: https://image.blog.chaosjohn.com/Debug-php/chrome-xdebug-helper-activation.png
[img13]: https://image.blog.chaosjohn.com/Debug-php/chrome-xdebug-helper-activation-fix.png
[img14]: https://image.blog.chaosjohn.com/Debug-php/ready-for-debug-short.png
[img15]: https://image.blog.chaosjohn.com/Debug-php/debug-start.png
[img16]: https://image.blog.chaosjohn.com/Debug-php/debug-point.png
[img17]: https://image.blog.chaosjohn.com/Debug-php/phpstorm-create-build-in-server-configuration.png
[img18]: https://image.blog.chaosjohn.com/Debug-php/phpstorm-create-build-in-server-configuration-finish.png
[img19]: https://image.blog.chaosjohn.com/Debug-php/phpstorm-run-build-in-configuration.png
[img20]: https://image.blog.chaosjohn.com/Debug-php/vscode-create-run-configuration.png
[img21]: https://image.blog.chaosjohn.com/Debug-php/vscode-php-debug-configurations.png
[img22]: https://image.blog.chaosjohn.com/Debug-php/vscode-php-debug.png
[img23]: https://image.blog.chaosjohn.com/Debug-php/phpstorm-remote-debug.png