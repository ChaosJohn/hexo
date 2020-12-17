---
title: IPv4/IPv6 双栈 ddns (DNSPod版)
date: 2020-12-16 19:25:06
tags: [ddns]
thumbnail: https://image.blog.chaosjohn.com/ddns-for-dnspod/banner.png
---

[欢迎转载，但请在开头或结尾注明原文出处【blog.chaosjohn.com】](https://blog.chaosjohn.com/ddns-for-dnspod.html)

## 前言
笔者很早以前就有一个梦想，即：我所有的联网设备，都要能随时随地访问到。比如在公司访问我家里的电脑和 NAS，以及在家里访问公司的工作电脑。

于是笔者开始了漫长的折腾之路：

1. 最开始，家里和公司都是电信宽带，我就不厌其烦地骚扰电信客服，申请公网IP。当路由器分配到了公网IP后，配合 `DMZ` 和 `端口映射`，就能将路由器下设备的某些端口暴露在公网上。缺点：如果设备和需要暴露到公网的服务比较多，不拿个记事本把 `什么服务暴露在什么端口` 记录下来，用到的时候总是会记混淆。
2. 后来搬家后宽带从电信换成了移动，获取公网IP就成了奢望（我还真上过工信部投诉，结果移动致电过来表示可以给我安排停服退款，我怂了，继续用吧）。然后就买了一个[花生棒](https://hsk.oray.com/device)，即花生壳的内网穿透版，因免费版只给两条映射，所以笔者只能贡献给常用机的 SSH(22端口) 和 VNC(5900端口)。缺点：奈何能用的端口只有两个，以及每月只有2G的流量，笔者撑了一段时间还是放弃了它。
3. 后来逐渐接触到各种内网穿透方案，先是 `ngork`，后来是 `frp`，将服务端部署在了自己的云主机上后，端口数量限制再也不是问题了，想配多少就配多少。缺点：国内云主机的带宽实在太贵，以阿里云为例，1M带宽23元每月，2M带宽46元，3M带宽71元，4M带宽96元，5M带宽125元，带宽越大单价越贵；国外云主机带宽虽然足，但是延迟太高了，真的很难取舍。
4. 再后来，又接触到了 [n2n](https://www.ntop.org/products/n2n/)，个人的云主机上搭建一个节点服务器 `supernode`，不同设备都连接 `supernode`，协商后直接进行点对点（P2P）通信，即真正的设备间数据传输是直连的而不经过 `supernode`。优点：全端口可用，速度不受限。缺点：因为是 `P2P` 方案，所以需要设备都安装客户端，但它把 `Windows` / `Linux` / `macOS` / `Android` 都支持遍了，就是没有 `iOS` 的客户端。
5. 17年下半年的时候，笔者突然发现了神器 `ZeroTier`。它就是一个增强版的 `n2n`，不同的是，它默认使用官方的节点服务器（当然个人也可以自建），更为重要的是，几乎所有的系统平台，它都支持，就连各类NAS也能安装使用。笔者直到今天，还在深度使用它。举个例子证明笔者有多爱它：官网是 `zerotier.com`，笔者于 **2018-01-12** 在腾讯云注册了域名 `zerotier.cn`，专门用来解析 `ZeroTier` 分配给设备的 `内网IP`，并续费至今。（如果用 `whois` 查一下该域名，还能发现笔者的真名和邮箱哦哈哈哈哈哈）缺点：在设备点对点之间的线路优化好之前，丢包比较严重，甚至连不上。但只要线路逐渐优化好之后，速度几乎能跑满带宽。
6. 19年下旬的时候，笔者突然发现，家庭宽带和手机蜂窝网络，居然都原生支持 `IPv6` 了，即无论是路由器下的设备，还是插了SIM卡的移动设备，都能分配到 `IPv6地址` 了。这意味着，通过某设备的 `IPv6` 地址，可以直接访问到该设备的所有端口（80/443端口是否被禁得看各地运营商政策）。

## **IPv6** 的问题
用 `IPv6` 实现所有设备的连通，这点非常棒，就只差一个问题需要解决：适用于 `IPv6` 的 `ddns` 方案。

什么是 `ddns` 呢？它的全称为 `dynamic dns`，即 `动态域名解析`。

使用场景：家庭宽带下，`IP地址` 会被运营商定期更换，在自身 `IP地址` 发生变化后，将新的IP地址提交给 `dns解析服务商`，让约定的域名解析更新为新的IP地址。

上面 `折腾之路[1]`，通过电信宽带分配的公网IP访问服务，也是需要用 `ddns` 来更新 `IP地址` 到自己的域名上，不过一般路由器都内置了 `ddns` 功能，傻瓜式配置起来也很简单。

但是：

- 这些成熟且内置的 `ddns` 方案，都只适用于 `IPv4`
- 对于 `IPv4`，局域网内只需有一台设备（可以是路由器本身，也可以是下属设备）配置 `ddns` 就可以了，因为整个局域网都共用一个公网IP；而对于 `IPv6`，每个设备都有自己的公网IP，即需要公网被访问的每台设备都要单独配置 `ddns`

## 解决 **IPv6** 的 **ddns**
因笔者的域名几乎都是在 `腾讯云` 上购买的，并且使用 `DNSPod` 进行解析，故以下所有的方案都是聚焦于 `DNSPod`。

### 第一版方案
去 DNSPod 开源社区寻找 `IPv6-ddns` 的解决方案。不得不说，很多工具都写的很棒，代码写的很漂亮，但是都不适用笔者，主要原因为，笔者的设备太多了，所搭载的系统也太杂了，导致那些工具即使为了提高兼容性，采用 `纯Python` 和 `纯Shell` 编写，也无法满足所有设备，比如 `ESXi`，这家伙虽然底层是 `Linux`，但是魔改了很多，无论是 `Shell` 还是 `Python` 环境，都是阉割过的，实测大部分工具都无法正常运行；另一方面，如果遇到多网卡（含虚拟网卡）的场景，那些工具很难获取到准确的且公网可达的本地 `IPv6地址`。

### 第二版方案
为了解决 *第一版方案* 的两个痛点：

1. 如何获取到准确的 `IPv6` 地址
2. 如何能在 *几乎* 所有系统上都适用（除开Windows，因为笔者几乎不用）

针对 `痛点1`：既然通过读取网卡信息无法做到准确地获取公网可达的 `IPv6地址`，那就更换思路，网上不是有很多查询自身 `IPv6地址` 吗，比如 `api6.ipify.org`
```
$ curl api6.ipify.org
240e:47b:1660:4939:881f:1234:5678:9abc
```

因为几乎所有的系统都预装了 `curl` 或者 `wget`（比如 `ESXi` 无 `curl` 但内置了阉割版的 `wget`），所以 `痛点1` 完美解决。

同时也给解决 `痛点2` 提供了思路：如果我在自己的云主机上部署一个服务，设备定期将自己的 `IPv6地址` 和与其绑定的域名通过 `curl` / `wget` 提交给该服务，让该服务在云主机上代理进行 `ddns`，将新解析提交给 DNSPod，不就解决了吗？

于是笔者：

1. 参考 `DNSPod API` 用 `NodeJS` 写了一个 `ddns代理` 的服务部署在云主机上（用 `NodeJS` 进行编写的原因仅仅是当时的工作内容和 `NodeJS` 相关，顺手而已）
2. 各设备上用 `cron` 或 `while + sleep` 定时（笔者设定的是1分钟）先从 `api6.ipify.org` 获取本机 `IPv6地址` 再向该 `ddns代理服务` 提交。

Perfect！

### 第三版方案
`第二版` 方案用的挺舒心的，但是运行了一段时间发现，出问题了，我好多设备都每分钟请求一次，还挺频繁的，导致 `api6.ipify.org` 限流，拒绝返回。

换用其他的 `IPv6地址查询服务`，也都一样，隔一段时间就可能被限流。

所以笔者萌生了自己写一个 `IPv6地址查询服务`，只为自己提供服务，限不限流我自己说了算！

这回为了造轮子能造的快点，放弃了 `NodeJS` 改用了 `PHP`，直接上代码：

`PHP` 代码
```
$ cat /home/chaos/IPChecker/index.php
<?php

echo $_SERVER['REMOTE_ADDR'];
```

`Nginx` 配置文件
```
$ cat /etc/nginx/conf.d/ip-check.conf
server {
  server_name ip.example.com ipv6.example.com;
  listen 80;
  listen [::]:80;

  root /home/chaos/IPChecker;
  location / {
    try_files $uri $uri/ /index.php;
    fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
  }
}
```

是不是很简单，`PHP` 代码就只有一行代码 `echo $_SERVER['REMOTE_ADDR'];`，即直接输出请求的来源地址，而且同时支持 `IPv4/IPv6`：

- `curl ip.example.com` 返回 `IPv4` 公网地址
- `curl ipv6.example.com` 返回 `IPv6` 公网地址

（注：该服务只能部署在有 `IPv6` 地址的云主机上，因为 `IPv6` 只能访问 `IPv6`）

### 第四版方案
完成一次 `ddns`，要先从 `ipv6.chaosjohn.com` 获取 `IPv6地址`，再向 `ddns代理服务` 提交，得两步操作，太麻烦了。

所以笔者决定把两项服务合二为一，依旧废话不多说，上代码：

项目根目录
```
$ ls -alh /home/chaos/ddns-dnspod
total 88K
drwxrwxr-x  4 chaos chaos 4.0K Nov  3 06:57 .
drwxr-xr-x 23 chaos chaos 4.0K Dec 16 15:30 ..
-rw-rw-r--  1 chaos chaos   75 Nov  2 09:24 composer.json
-rw-rw-r--  1 chaos chaos 2.7K Nov  2 09:24 composer.lock
-rw-rw-r--  1 chaos chaos   71 Nov  3 01:46 config.ini
-rw-rw-r--  1 chaos chaos   53 Nov  3 05:54 config-sample.ini
drwxrwxr-x  8 chaos chaos 4.0K Nov  3 06:57 .git
-rw-rw-r--  1 chaos chaos  524 Nov  3 05:57 .gitignore
-rw-rw-r--  1 chaos chaos 2.7K Nov  3 02:59 index.php
-rw-rw-r--  1 chaos chaos  35K Nov  3 06:02 LICENSE
-rw-r--r--  1 chaos chaos  358 Nov  2 14:04 nginx.conf
-rw-r--r--  1 chaos chaos  362 Nov  3 05:53 nginx-sample.conf
-rw-rw-r--  1 chaos chaos  941 Nov  3 06:57 README.md
drwxrwxr-x  4 chaos chaos 4.0K Nov  2 09:24 vendor
```

配置文件（其中 `token` 是 `DNSPod` 密钥管理里的 `ID,token`，形如 `30345,ac0000000918368b1cfa16f4fc6e28cd`）
```
$ cat /home/chaos/ddns-dnspod/config.ini
[config]
token = 'YOUR-DNSPOD-TOKEN'
key = 'YOUR-KEY'
```

`PHP` 代码
```
$ cat /home/chaos/ddns-dnspod/index.php
<?php

require __DIR__ . '/vendor/autoload.php';

use Curl\Curl;
$curl = new Curl();

$ini_array = parse_ini_file('config.ini', true);

$token = $ini_array['config']['token']; // dnspod token
$required_key = $ini_array['config']['key'];

//echo $_SERVER['HTTP_X_REAL_IP'];
$address = $_SERVER['REMOTE_ADDR'];
$isIPv6 = strpos($address, ':') > -1;
$type = $isIPv6 ? 'AAAA' : 'A';
$domain = $_GET['domain'];
$sub = $_GET['sub'];
$key = $_GET['key'];

if (!$domain || !$sub || 0 != strcmp($key, $required_key)) exit;

$output = [
  'domain' => $domain,
  'sub' => $sub,
  'type' => $type,
  'address' => $address
];

$form = "login_token=${token}&format=json&domain=${domain}&sub_domain=${sub}&record_type=${type}&record_line_id=0";
$response = $curl->post('https://dnsapi.cn/Record.List', $form);
if ($curl->error) {
  $output['msg'] = 'Record.List Error' . $curl->errorCode . ': ' . $curl->errorMessage . "\n";
} else {
  $records = $response->records;  
  if (null == $records) $records = [];
  $records = array_filter($records, function($record) use($type) { 
    return 0 == strcmp($record->type, $type); 
  });
  
  if (count($records) > 0) { // if record exists
    $record = $records[0];
    if (0 == strcmp($record->value, $address)) { // skip if same
      $output['msg'] = 'Same record. Skipped';
      echo json_encode($output);
      exit;
    }

    // update record
    $record_id = $record->id;
    $form = "login_token=${token}&format=json&domain=${domain}&record_id=${record_id}&sub_domain=${sub}&value=${address}&record_type=${type}&record_line_id=0";
    $response = $curl->post('https://dnsapi.cn/Record.Modify', $form);
    if ($curl->error) {
      $output['msg'] = 'Record.Modify Error' . $curl->errorCode . ': ' . $curl->errorMessage . "\n";
    } else {
      if (null != $response->status && null != $response->status->code && '1' == $response->status->code) {
        $output['msg'] = 'Modification succeeded';
      } else {
        $output = $response;
        $output->action = 'Record.Modify';
      }
    }
  } else { // record not exists => create 
    $form = "login_token=${token}&format=json&domain=${domain}&sub_domain=${sub}&record_type=${type}&record_line_id=0&value=${address}";
    $response = $curl->post('https://dnsapi.cn/Record.Create', $form);
    if ($curl->error) {
      $output['msg'] = 'Record.Create Error' . $curl->errorCode . ': ' . $curl->errorMessage . "\n";
    } else {
      if (null != $response->status && null != $response->status->code && '1' == $response->status->code) {
        $output['msg'] = 'Creation succeeded';
      } else {
        $output = $response;
        $output->action = 'Record.Create';
      }
    }
  }
}
echo json_encode($output, JSON_UNESCAPED_UNICODE);
```

`Nginx` 配置文件
```
$ cat /etc/nginx/conf.d/ddns-dnspod.conf
server {
  server_name ddns.example.com ddns6.example.com;
  listen 80;
  listen [::]:80;

  root /home/chaos/ddns-dnspod;
  location / {
    try_files $uri $uri/ /index.php;
    fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
  }
}
```

具体使用：

- 有 `cron` 的话，在 `crontab` 里添加一行：
```
* * * * * logfile='/tmp/ddns6.log'; echo "\n$(date)" >> $logfile && curl 'http://ddns6.example.com?key=YOUR-KEY&domain=DOMAIN&sub=SUBDOMAIN' >> $logfile
```
- 无 `cron` 的话，使用 `while + sleep`
```
while true; do logfile='/tmp/ddns6.log'; echo "\n$(date)" >> $logfile && curl 'http://ddns6.example.com?key=YOUR-KEY&domain=DOMAIN&sub=SUBDOMAIN' >> $logfile; sleep 1; done
```
- 如果没有 `curl`，只有 `wget` 的话，将 `curl [url]` 替换为 `wget -q -O - [url]`


代码笔者已经上传到 `Github`，如何部署请移步 [ChaosJohn/ddns-dnspod](https://github.com/ChaosJohn/ddns-dnspod) 参考哈（喜欢的话给个 Star 呗）

## 最后
`所有联网设备都可随时随地访问` 这个梦想最终通过 `IPv6` 完美达成了。

期间经历了多个方案，终于在 `折腾之路` 上告一段落了。

如果后期有更优解，笔者还会回来的！

---

最后，如果该文对读者有些许帮助，考虑下给点捐助鼓励一下呗😊
![](https://image.blog.chaosjohn.com/donate-me.png)