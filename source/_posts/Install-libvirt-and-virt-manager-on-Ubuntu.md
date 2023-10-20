---
layout: post
title: Ubuntu 安装libvirt / virt-manager
date: 2023-10-20 22:28:50
tags: [ubuntu, Virtualization, libvirt, virt-manager]
thumbnail: Install-libvirt-and-virt-manager-on-Ubuntu/ubuntu.png
---

[欢迎转载，但请在开头或结尾注明原文出处【blog.chaosjohn.com】](https://blog.chaosjohn.com/Install-libvirt-and-virt-manager-on-Ubuntu.md)

两者的区别：

- libvirt是个虚拟化平台
- virt-manager是libvirt的管理工具（安装virt-manager会默认安装libvirt）

在18.04以及之前版本，安装libvirt，只需要安装**libvirt-bin**

```bash
$ sudo apt install libvirt-bin
```

但是在18.10，libvirt-bin已经被拆分成两部分**libvirt-daemon-system**和**libvirt-clients**

```bash
$ sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils
```

如果只想安装virt-manager，但又不想安装本地的虚拟化平台，可以执行

```bash
$ sudo apt install virt-manager ssh-askpass-gnome --no-install-recommends
```

---
参考资料：
- [libvirt](https://libvirt.org/) / [virt-manager](https://virt-manager.org/)
- [KVMVirtManager on Ubuntu](https://help.ubuntu.com/community/KVM/VirtManager)
- [Virtualisation tools](https://ubuntu.com/server/docs/virtualization-virt-tools)
  - virt-manager (Virtual Machine Manager)
  - virt-viewer (Virtual Machine Viewer)
  - virt-install
  - virt-clone
- [Running virt-manager and libvirt on macOS](https://www.arthurkoziel.com/running-virt-manager-and-libvirt-on-macos/)
  ```bash
  # 安装libvirt([qemu在2018年提供了对Hypervisor.Framework的支持](https://wiki.qemu.org/ChangeLog/2.12))
  $ brew install libvirt
  $ brew services start libvirt

  # 安装virt-manager
  $ brew install virt-manager
  $ virt-manager --connect="qemu+ssh://chaos@ubuntu-w510.jeek.club/system?socket=/var/run/libvirt/libvirt-sock"
  ```
- [Using QEMU to create a Ubuntu 20.04 Desktop VM on macOS](https://www.arthurkoziel.com/qemu-ubuntu-20-04/)
- [Remote virt-manager from Mac OS @gist](https://gist.github.com/davesilva/da709c6f6862d5e43ae9a86278f79188)
  ```bash
  # Step 1: Allow your user non-root access to KVM
  ## SSH to the Linux machine and add your SSH user to the libvirt group
  $ sudo usermod -a -G libvirt $(whoami)

  ## Running id $(whoami) should list the libvirt group. You may also need to edit /etc/libvirt/libvirtd.conf to ensure that the socket has the right owner and permissions. Make sure you have these lines in there:
  unix_sock_group = "libvirt"
  unix_sock_rw_perms = "0770"

  # Step 2: Install virt-manager for Mac OS
  $ brew install virt-manager

  # Step 3: Connect virt-manager to the Linux machine over SSH
  ## If you try to tell virt-manager to connect over SSH by just specifying the hostname it might not work though. You'll see an error like End of file while reading data: nc: unix connect failed: No such file or directory: Input/output error. The file it's referring to is the libvirtd socket and the reason it can't find it is because it's looking in the wrong place. It's looking for the sock file in /usr/local/lib but it actually lives at /var/run/libvirt/libvirt-sock (at least it does on Debian). Luckily there's a query param you can pass in the SSH URL to override the socket location:
  $ virt-manager --connect="qemu+ssh://USER@HOSTNAME/system?socket=/var/run/libvirt/libvirt-sock"
  ```
- [《统信UOS》UOS系统安装KVM虚拟机](https://knowledge.ipason.com/ipKnowledge/knowledgedetail.html/1347) → 最原始需求
  ```bash
  $ sudo apt install virtinst python-libvirt virt-viewer virt-manager bridge-utils uml-utilities ovmf qemu-efi libvirt-daemon-system libvirt-clients libvirt-daemon qemuctl qemu-utils qemu-user qemu-system qemu qemu-system-common qemu-system-gui
  ```

---

最后，如果该文对读者有些许帮助，考虑下给点捐助鼓励一下呗😊
![](hello-world/donate-me.png)
