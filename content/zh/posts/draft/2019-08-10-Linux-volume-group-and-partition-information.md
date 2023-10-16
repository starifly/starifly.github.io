---
title: Linux磁盘分区和卷组信息
author: starifly
date: 2019-08-10T11:06:53+08:00
categories: [linux]
tags: [linux,磁盘,分区]
draft: true
slug: Linux-volume-group-and-partition-information
---

## 分区

### 通过lsblk命令查看分区表

`lsblk`命令列出系统的所有块设备及其逻辑分区。在终端中输入`lsblk`命令以列出分区表：

![](https://starifly.github.io/images/lsblk.jpg)

- NAME - 设备名称
- MAJ:MIN -Major和Min Device number
- RM-设备是否可拆卸（1）或不可拆卸（0）
- SIZE - 设备大小
- RO -设备是只读的（1）还是不是（0）
- TYPE - 设备类型，即，如果它是磁盘或分区等。
- MOUNTPOINT - 设备的安装点（如果适用）。

其中 `sda` 是磁盘设备，`sda1` 和 `sda2` 是分区。

### 使用fdisk命令获取分区列表

`fdisk` 命令主要用于磁盘分区和格式化，但是，在这里我们将使用它来列出分区表，方法是使用特定的标志。

`-l` 标志与 `fdisk` 一起使用以列出指定设备的分区表，然后退出。 如果未提及任何设备名称， `fdisk` 将使用 ``/proc/partitions` 文件中提到的设备。如下所示：

![](https://starifly.github.io/images/fdisk.jpg)

### 使用sfdisk命令查看分区

虽然 `sfdisk` 命令主要用于操作 `Linux` 上的分区表，但它也可以用于通过使用以下语法列出设备的分区表：

`sudo sfdisk -l/dev/devicename`

例如：

`sudo sfdisk -l /dev/sda`

### 使用parted命令获取硬盘分区

列出设备分区表的另一种方法是通过 `parted`命令。 `parted`d命令在前面提到的 `fdisk` 和 `sfdisk` 命令上有优势，因为前者没有列出大小超过2 TB的分区。

使用以下语法查看设备的分区表：

`sudo parted /dev/devicename`

例如：

`sudo parted /dev/sda`

该命令将进入“（parted）”提示模式。 您可以在此处输入以下值，以帮助您查看设备的分区表。

- Unit GB：通过此输入，您可以选择以GB显示的输出。
- Unit TB：通过此输入，您可以选择要在TB中显示的输出。

输入您的选择，之后系统将显示相应的分区表。

输入 `help` 命令，会列出所有可用的命令。常用的是 `cp`， `rm`， `resize`， `set`， `mkpartfs`， `print`。

1） `print` 用于显示当前的分区情况

2） `set` 可以设置分区的标志： `set 1 boot on`

3） `mkpartfs` 创建分区： `mkpartfs primary linux-swap 1KB 2MB`

4） `rm` 删除分区，可用 `resure` 恢复

5） `cp` 将拷贝分区内容到新的分区

6） `resize` 可以改变分区的大小

实际的应用场景：无损压缩大分区

用 `resize` 可以修改分区的大小，但是要做到无损，只能减小该分区的结束位置，因为分区表的信息在起始的位置。但是如何知道，该分区已经占用了多少空间。可以用 `df` 命令来查看：有一项是 `available` ，注意不能用总容量 `-used` 部分计算，原因就不说了吧。这样 `resize` 可以保证无损压缩。

注意使用前，要先 `unmount` 该分区。交换分区要 `swapoff` ，才能修改。修改完后用 `swapon` 打开， `swapon -s` 可以显示交换分区使用情况。

要退出 `parted` 命令模式，只需键入 `quit` ，然后单击 `Enter` 。

或者，您可以使用以下命令列出系统所有块设备上的所有分区布局:

`sudo parted -l`

## 卷组

`vgs`， `vgscan`， `vsdisplay` 查看卷组信息。

`pvs`， `pvscan`， `pvdisplay` 查看物理卷信息。

`lvs`， `lvscan`， `lvdisplay` 查看逻辑卷信息。
