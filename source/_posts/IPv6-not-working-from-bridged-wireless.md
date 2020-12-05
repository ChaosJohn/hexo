---
title: IPv6在虚拟机通过无线网卡桥接的网络下无法使用(VMware WorkStation)
date: 2020-11-28 18:19:22
tags: [Virtualization, IPv6, network, VMware]
thumbnail: https://image.blog.chaosjohn.com/IPv6-not-working-from-bridged-wireless/vmware.jpg
---

[欢迎转载，但请在开头或结尾注明原文出处【blog.chaosjohn.com】](https://image.blog.chaosjohn.com/IPv6-not-working-from-bridged-wireless)

## 背景
今年年中的时候，在油管上看到 *悟空大大* 关于 [Minisforum GK41](https://store.minisforum.com/products/minisforum-gk41-mini-pc) 的视频，心里痒痒，于是也入手了一个。主要具体参数为：
- CPU: Intel® Celeron® Processor J4125
- 内存: LPDDR4 8GB (On Board)
- 网络: 双千兆螃蟹卡 + 802.11ac双频Wifi

买之前想的很美妙，买它！当软路由！就奔着它双网口且不到日常使用5~6W的超低功耗！

结果买回来就开始纠结了，用它当软路由是不是太浪费了？毕竟128GB的M.2固态，只拿来跑OpenWRT，很难把空间占满啊；内部也没有SATA接口，无法把废弃硬盘利用起来；也不支持从TF卡槽启动，手里一堆TF卡也无用。

机子买来就预装了正版Windows10，想着好久没有用Windows了，那就顺便玩玩吧。于是手就不听使唤的把 `VMware WorkStation`给安装上了，又不听使唤的装了个 `Manjaro Linux` 虚拟机。

## 问题：客户机IPv6无法使用
网络拓扑结构：
- GK41主机通过无线网卡连接Wi-Fi
- 虚拟客户机采用桥接模式（非NAT模式），桥接无线网卡加入网络

至此，虚拟客户机也从上级路由器获得IPv4地址，与GK41主机处于同一个局域网。

但是无法访问IPv6网络（由于这台GK41已经转手卖掉了很久，所以记不清是无法从上级路由器获取IPv6地址还是分配到了IPv6地址但无法联通网络）。因笔者将GK41放在公司，脱离了IPv6就意味着笔者无法便捷地从家里访问到这台虚拟客户机。

## 解决过程（最后发现无解）
尝试：插上网线，把桥接模式从无线网卡改为有线网卡，IPv6奇迹般的有了！所以问题肯定出在了 *无线桥接* 上。

于是笔者各种查资料爬帖，各种折腾，最后发现了2008年的一篇帖子[IPv6+Bridged+Wireless](https://communities.vmware.com/t5/VMware-Fusion-Discussions/IPv6-Bridged-Wireless/m-p/2038236/highlight/true#M124911)。

文中描述 `This is a known issue, IPv6 is not expected to work over a wireless bridge.` 在帖子发布的时候，这个bug（编号#26078）覆盖了VMware的全线产品，包括WorkStation/Player/Fusion。[另参考](http://christophe.vandeplas.com/2008/11/vmware-network-bridge-over-wireless_6176.html)

（实测 *VMware Fusion* 在很早以前就已经修复该bug，因为笔者有一台2009年的MacBook，还跑着 *Mac OS X 10.7 Lion*，运行着 *VMware Fusion 6.0.6版本*，而其内的虚拟客户机的IPv6是正常的，包括后来的高版本Fusion下的虚机IPv6都好使。所以很明显，只是WorkStation上这个bug依旧存在）

至此，放弃折腾，Over。(如何有网友发现解决该问题的办法，不吝赐教哈，评论区或者加我微信 *Chaos_John* )
