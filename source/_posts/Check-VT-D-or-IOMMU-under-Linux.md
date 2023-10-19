---
title: Linux 下检查 VT-d / IOMMU 是否开启
date: 2020-12-15 20:43:47
thumbnail: Check-VT-D-or-IOMMU-under-Linux/banner.png
---

[欢迎转载，但请在开头或结尾注明原文出处【blog.chaosjohn.com】](https://blog.chaosjohn.com/Check-VT-D-or-IOMMU-under-Linux.html)

## 介绍
`VT-d` 和 `IOMMU` 其实都是指的 `I/O 虚拟化`，只不过前者是 `Intel` 的叫法，后者是 `AMD` 的叫法：

- `VT-d` 全称为 `Intel® Virtualization Technology for Directed I/O`
- `IOMMU` 全称为 `Input/Output Memory Management Unit`

这项技术是可以让PCI-e设备的资源直接分配给虚拟机，即 `PCI直通`。

举个例子，在虚拟客户机里可以直接访问到物理显卡，性能比使用由 `VMM/Hypervisor` 虚拟出来的显卡好很多，并且还支持 `显卡加速`。

## 在 Linux 下检查是否开启
一般来说，在主板的 `BIOS/UEFI` 里，能找到 `VT-d / IOMMU` 的设置项，设为开启即可。

但是也有特殊情况，某些主板里是找不到该项设置的，这里分两种情况：

- 主板硬件或固件不支持 `I/O 虚拟化`
- 主板刷入了阉割版的固件，但是实际上 `VT-d / IOMMU` 是被启用的

那如果在 `Linux` 下如何检查是否开启呢？

如果 `VT-d / IOMMU` 被启用，`Linux` 在启动过程中会配置 `DMA重映射`，所以简单的方法是在 `dmesg` 里查找 `DMAR` 相关项。

- 在已开启的机子上：
```
# dmesg | grep DMAR
[    0.000000] ACPI: DMAR 0x00000000BBECB000 0000A8 (v01 LENOVO TP-R0D   00000930 PTEC 00000002)
[    0.001000] DMAR: Host address width 39
[    0.001000] DMAR: DRHD base: 0x000000fed90000 flags: 0x0
[    0.001000] DMAR: dmar0: reg_base_addr fed90000 ver 1:0 cap 1c0000c40660462 ecap 19e2ff0505e
[    0.001000] DMAR: DRHD base: 0x000000fed91000 flags: 0x1
[    0.001000] DMAR: dmar1: reg_base_addr fed91000 ver 1:0 cap d2008c40660462 ecap f050da
[    0.001000] DMAR: RMRR base: 0x000000bbdd8000 end: 0x000000bbdf7fff
[    0.001000] DMAR: RMRR base: 0x000000bd000000 end: 0x000000bf7fffff
[    0.001000] DMAR-IR: IOAPIC id 2 under DRHD base  0xfed91000 IOMMU 1
[    0.001000] DMAR-IR: HPET id 0 under DRHD base 0xfed91000
[    0.001000] DMAR-IR: Queued invalidation will be enabled to support x2apic and Intr-remapping.
[    0.002000] DMAR-IR: Enabled IRQ remapping in x2apic mode
```
观察最后一行，可以看到类似 `DMAR-IR: Enabled IRQ remapping in xxxxx mode` 的输出

- 在未开启的机子上：
```
# dmesg | grep DMAR
#
```
什么输出都没有，即没有找到任何与 `DMAR` 相关的项

---

最后，如果该文对读者有些许帮助，考虑下给点捐助鼓励一下呗😊
![](hello-world/donate-me.png)