---
layout: post
title: Ubuntu 22.04 单网卡创建br0(免断网)
date: 2023-10-20 22:12:20
tags: [ubuntu, network, bridge]
thumbnail: Create-br0-without-losing-connection-under-Ubuntu-22-04/ubuntu.png
---

[欢迎转载，但请在开头或结尾注明原文出处【blog.chaosjohn.com】](https://blog.chaosjohn.com/Create-br0-without-losing-connection-under-Ubuntu-22-04.md)

在译文《[**通过 libvirt 和 KVM 使用桥接网络**](https://www.zml.me/post/316051871548702720)》（原文<[How to use bridged networking with libvirt and KVM](https://linuxconfig.org/how-to-use-bridged-networking-with-libvirt-and-kvm)>）中，提到了如何用`ip`命令创建桥接网卡br0，以及如何用virsh命令将br0同步给libvirt虚拟机知道。

```bash
# 创建新网桥br0
$ sudo ip link add br0 type bridge
# 验证br0是否创建成功
$ sudo ip link show type bridge
```

```bash
# 启用额外的物理网卡
$ sudo ip link set enp0s29u1u1 up
# 将网卡添加到br0
$ sudo ip link set enp0s29u1u1 master br0
```

这边我们需要注意，创建br0没啥问题，但是将物理网卡添加到br0，是需要引起注意的。原文中提到：

> 注意，在这种情况下，你不能使用你的主以太网接口，因为一旦它被添加到网桥，你就会失去连接，因为它将失去它的 IP 地址。在这种情况下，我们将使用一个额外的接口，enp0s29u1u1：这是一个由连接在我的机器上的以太网转USB适配器提供的接口。
> 

所以，如果是单网卡的主板，且没有额外的网卡（USB或PICe），这样的配置是非常危险的，因为系统会立马断网，且无法自动恢复。

但也不用很担心系统挂了，因为配置没有持久化（持久化在原文中有写，但不适用于改用了netplan的Ubuntu），只需重启系统即可恢复。

那怎么办呢？

我想到我另一台rk3356的ARM单板，正运行着ProxmoxVE，单物理网卡但同时每台虚拟机/容器都是从上级路由器直接获取IP地址，就是通过vmbr0桥接了物理网卡

```bash
$ cat /etc/network/interfaces
auto lo
iface lo inet loopback

iface eth0 inet manual

auto vmbr0
iface vmbr0 inet static
        hwaddress ether 42:87:5c:c9:b8:12
        address 10.1.1.190/24
        broadcast 10.1.1.255
        netmask 255.255.255.0
        gateway 10.1.1.1
        bridge-ports eth0
        bridge-stp off
        bridge-fd 0
        dns-nameservers 10.1.1.1
```

参考这个方案改在Ubuntu上不适用，因为找不到`/etc/network/interfaces`文件，Ubuntu 18.04 开始改用了netplan

经过几番搜索，在另一篇文章中《[kvm单网卡桥接模式](https://aoyouer.com/posts/kvm-bridge/)》，发现了适用于netplan的配置方式

```bash
$ vim /etc/netplan/01-network-manager-all.yaml
network:
  ethernets:
    enp125s0f0:
      dhcp4: no
  bridges:
    br0:
      interfaces: [enp125s0f0]
      dhcp4: no
      addresses: [10.1.1.168/24]
      gateway4: 10.1.1.1
      parameters:
        stp: true
        forward-delay: 4
      nameservers:
        addresses: [10.1.1.1]
  version: 2

$ sudo netplan apply
```

然后用新创建的br0给虚拟机定义一个新的“网络”

```bash
# 任意路径创建文件
$ vim virsh-br0-network.xml
<network>
    <name>br0-network</name>
    <forward mode="bridge" />
    <bridge name="br0" />
</network>

# 通过virsh定义新的桥接网络
$ sudo virsh net-define virsh-br0-network.xml

# 激活新网络
$ sudo virsh net-start br0-network

# 使其自动启动
$ sudo virsh net-autostart br0-network

# 查看网络列表
$ sudo virsh net-list --all
 Name          State    Autostart   Persistent
------------------------------------------------
 br0-network   active   yes         yes
 default       active   yes         yes
```

然后libvirt就能用这个br0桥接网卡，从上级路由器获取IP地址并且联网了，而不是`NAT`模式。

---
参考资料:
- [通过 libvirt 和 KVM 使用桥接网络](https://www.zml.me/post/316051871548702720) / [How to use bridged networking with libvirt and KVM](https://linuxconfig.org/how-to-use-bridged-networking-with-libvirt-and-kvm)

- [kvm单网卡桥接模式](https://aoyouer.com/posts/kvm-bridge/) ⇒ netplan

- [KVM虚拟机网络配置 Bridge方式，NAT方式](https://zhuanlan.zhihu.com/p/359864370)  / [linux中KVM桥接网卡br0](https://developer.aliyun.com/article/530814) / [Linux 下如何配置桥接网卡，可以实现主机与虚拟机互通？](https://v2ex.com/t/654887)/ [转: Linux下单网卡多vlan多虚拟机](https://www.cnblogs.com/itfriend/archive/2012/05/30/2526660.html)  ⇒ /etc/sysconfig/network-scripts/ifcfg-br0

- [how to create sub-interface](https://askubuntu.com/questions/1432883/how-to-create-sub-interface) ⇒ netplan

- [KVM Bridged Network Not Working](https://askubuntu.com/questions/179508/kvm-bridged-network-not-working) / [OpenStack Neutron单网卡桥接模式访问外网](https://www.cnblogs.com/openstackteam/p/5519961.html) ⇒ /etc/network/interfaces

- [Linux虚拟网络设备之bridge(桥)](https://segmentfault.com/a/1190000009491002)

---

最后，如果该文对读者有些许帮助，考虑下给点捐助鼓励一下呗😊
![](hello-world/donate-me.png)
