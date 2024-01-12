---
date: 2023-03-24
title: 蛟龙16K Linux 内置键盘失灵问题
tags:
- Linux
- 键盘
description: 锐龙笔记本 Linux 键盘失灵
---
# 蛟龙16K Linux 内置键盘失灵问题

## 问题环境

机型：蛟龙16K Jiaolong16K Series GM6BG0Q

本机型在6.0版本及之后的Linux内核上出现内置键盘失灵问题，进入系统后键盘毫无反应，加载evbug模块后可以在dmesg中看到键盘发送的键值错乱毫无意义。

我最初在安装archlinux时发现此问题，此时archlinux内核版本为6.2，我又尝试了ubuntu的安装镜像，结果惊讶的发现Ubuntu 22.04和22.10均无问题，这两个镜像分别使用了5.15和5.19内核，因此我又下载了ubuntu 23.04预览版本，此版本内核为6.2，键盘无效。因此可以定位到在linux 6.0到6.2之间的某个版本引入了此问题。

因此，我针对关键字Linux AMD Ryzen Keyboard进行了搜索，发现了在archlinux论坛中报告了Thinkbook 6800H版本出现了键盘无效问题^[Keyboard problem - Thinkbook 14+ ARA](https://bbs.archlinux.org/viewtopic.php?id=277260)，并在Linux 6.0中此问题得到了修复。

其中archlinux论坛用户@981213提到该机型的键盘设计比较奇怪，如下。
```
It's a weird design choice instead of a bug:
They made the keyboard IRQ active-low instead of the conventional active-high found in almost all other computers.
The kernel decided to override this, which made the keyboard controller non-functional.
```

为了解决这一问题，linux内核在补丁^[ACPI: skip IRQ override on AMD Zen platforms](https://lkml.kernel.org/linux-acpi/20220712020058.90374-1-gch981213@gmail.com/)中
改变了之前的行为，不再为AMD锐龙平台下的键盘中断的active-low改成active-high。

在这个补丁之前，内核对键盘中断做了特殊处理，对于active-low的中断，即认为是BIOS有Bug，将键盘中断替换为active-high。这种做法导致部分锐龙笔记本键盘失效，如上述论坛中讨论的机型，因为这些笔记本的键盘本身就是active-low的。

然而，蛟龙16K的BIOS的ACPI DSDT表里的键盘中断描述有误，DSDT中的键盘中断描述为active-low，然而机器键盘实际是active-high，因此在应用了这个补丁之后，蛟龙16K的键盘反而不能正常运行了。

为解决这一问题，我参考了这一文章^[Redmibook Pro 15 2022 锐龙版 的Linux驱动](https://zhuanlan.zhihu.com/p/530643928)中的方法，导出并修改了DSDT表中的描述，制作了dsdt补丁，从而修复了问题。

虽然此方法能够解决这一问题，但是仍然是不彻底的，只有BIOS更新才能彻底解决问题，然而机械革命客服表示不对linux系统下的问题提供支持，因此目前只能通过对自己机型做一个专门的ACPI patch来缓解这一问题。

