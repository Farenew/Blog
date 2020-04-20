---
date: '2019-03-17'
title: 配置Openwrt作为NAS来共享文件
author: ForenewHan
tags: ["代码", "硬件"]
categories: ["计算机"]
# external_link: 'https://forenewhan.science/programming/openwrt_NAS/'
math: true
slug: openwrt_NAS
---

# 配置Openwrt作为NAS来共享文件

多亏浩南哥的教程, 宿舍最近搞了个OpenWrt来绕过校园网的限制。最近看着这台路由器, 想着都刷了openwrt了, 不加点功能实在有些亏, 装个梯子吧感觉很麻烦(主要是我的梯子都是ssr的), 所以想着弄个NAS好了。这样就可以躺在床上用ipad看电影了, 而且还能做远程下载, 以后直接传个种子过来让路由器自动下载岂不美滋滋。不过可惜宿舍网不支持ipv6, 而我的资源基本都是来源于ipv6的pt站, 所以这部分就不配置了, 简单弄个NAS好了。教程如下:

## 1. openwrt支持usb设备

### 1.1. 配置USB的支持

openwrt相当于就是一个Linux发行版了，一个linux系统没理由不支持usb设备。openwrt也一样，但是需要安装一些对应的驱动等。

首先[安装block-mount](https://oldwiki.archive.openwrt.org/doc/techref/block_mount)，这个允许我们可以在openwrt上挂在block设备：

```sh
opkg update
opkg install block-mount
```

接下来安装openwrt的USB支持(1)：

```sh
opkg update
opkg install kmod-usb-storage
opkg install kmod-usb2
```

接下来安装对应的kernel modules, 这个是为了支持不同的文件系统。一般来说，U盘都是FAT32文件系统，但是openwrt有各种不同的系统支持，对于FAT32的文件系统安装：

```sh
opkg install kmod-fs-vfat
```

此外，可能会存在USB驱动无法识别的问题，推荐安装[`usbutils`](https://openwrt.org/packages/pkgdata/usbutils):

```sh
opkg install usbutils
```

但是FAT32系统存在不支持大文件的问题，只能支持4GB以下的文件，对于现在常见的一些`.mkv`格式的电影，往往都7,8GB，就比较棘手了(2), 这里大家常用的是`exFAT`, 但很不巧, openwrt并没有官方的支持。

这里做完后, 推荐重启一下路由器。重启后应该可以在控制界面中的system里看到一个mount point的选项。

### 1.2. 挂载USB设备

之后, 就可以插入U盘, 运行:`dmesg`, 在最下面应该有类似这样的信息, 这样的信息出现, 说明检测到了U盘的插入并可以正确识别:

```
[15844.578984] scsi 1:0:0:0: Direct-Access     Kingston DataTraveler 3.0 PMAP PQ: 0 ANSI: 6
[15844.591303] sd 1:0:0:0: [sdb] 60555264 512-byte logical blocks: (31.0 GB/28.9 GiB)
```

如果没能识别, 根据这里的log, 查看是否前面的内容都做到了。如果确定都做了, 且U盘未损坏, 那么参考[这里](https://openwrt.org/docs/guide-user/storage/usb-drives-quickstart)的要求有没有满足

这里安装了`usbutils`的话, 可以使用`lsusb`命令查看USB 设备信息:

```
root@OpenWrt:~# lsusb
Bus 001 Device 007: ID 0951:1666 Kingston Technology DataTraveler 100 G3/G4/SE9 G2
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub

root@OpenWrt:~# lsusb -t
/:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=ehci-platform/1p, 480M
    |__ Port 1: Dev 7, If 0, Class=Mass Storage, Driver=usb-storage, 480M
```

直接输入, 可以看到总线和接入的设备。带`-t`的命令则可以看到详细的信息:

- Bus..: 代表了主机芯片对应的线
- 这里ehci代表了USB 2.0, xhci是for USB3.0,uhci或者ohci代表了USB1.1
- Class=Mass Storage是说这个设备是USB设备
- Driver=usb-storage是指这个是[Bulk only Transport](https://en.wikipedia.org/wiki/USB_mass_storage_device_class), 一种通信协议。如果这一项是usb-storage-uas, 那么就是[USB Attached SCSI](https://en.wikipedia.org/wiki/USB_Attached_SCSI)协议。前者即USB 2.0, 各种常见外部硬件的通信协议。后者是USB 3.0。
- 最后的480M指的是协议速度是480mbps。

运行`block detect`, 应该可以看到连接的block设备的配置信息:

```
config 'global'
	option	anon_swap	'0'
	option	anon_mount	'0'
	option	auto_swap	'1'
	option	auto_mount	'1'
	option	delay_root	'5'
	option	check_fs	'0'

config 'mount'
	option	target	'/mnt/sdb1'
	option	uuid	'54D6-06CF'
	option	enabled	'0'
```

写到` /etc/config/fstab`里, 并且把其中的mount下的`option	enabled	'0'`中的0改成1

```
block detect > /etc/config/fstab
```

在前面配置文件对应的地方, `/mnt/sdb1`创建文件夹:

```sh
mkdir /mnt/sda1
```

然后挂载:

```sh
block mount
```

就可以使用了。

但是这一步挂载不一定能成功, 取决于文件系统。这个挂载是按照前面的配置文件对block设备进行挂载, 但常见的FAT32并不完全支持这种挂载(官方文档上并没有更新最新的[验证信息](https://oldwiki.archive.openwrt.org/doc/techref/block_mount))。

因此需要手动挂载, 首先确定位置, U盘的设备位置应该是在`/dev`下, 名字是`sda1`或者`sdb1`, 和前面的配置文件应该是对应的, 这里可以通过ls查看:

```sh
ls -l /dev/sd*
```

用`block`命令也可查看:

```sh
block info | grep "/dev/sd"
```

这里就可以看到详细的信息:

```
root@OpenWrt:~# block info
/dev/mtdblock8: UUID="495333694" VERSION="1" TYPE="ubi"
/dev/ubiblock0_0: UUID="82341949-641784c1-6b009cdb-1ff521e6" VERSION="4.0" MOUNT="/rom" TYPE="squashfs"
/dev/ubi0_1: UUID="58d776df-8cc4-409a-8737-ae43f3c89b5c" VERSION="w4r0" MOUNT="/overlay" TYPE="ubifs"
/dev/sdb1: UUID="54D6-06CF" LABEL="NO NAME" VERSION="FAT32" TYPE="vfat"
```

我这里是设备`sdb1`。

然后就把设备, 挂载到位置即可:

```sh
mount -t vfat /dev/sdb1 /mnt/sda1/
```

之后去`/mnt`下对应的地方查看, 就应该能打开U盘的文件了。

## 2. 配置NAS

这里使用Samba来实现这个方便快捷的分享软件。Samba是一个分享SMB(Server Message Block，服务器消息块)协议的自由软件, 已经成为了这一协议中Linux系统用的标准。

Samba提供了非常多的[功能](https://openwrt.org/docs/guide-user/services/nas/samba), 详细可以参考[文档](https://openwrt.org/docs/guide-user/services/nas/samba_configuration), 但这里仅用最基本的配置, 直接安装图形界面来简化操作。

首先安装Samba:

```sh
opkg update
opkg install samba36-server
```

然后要选择安装服务端, 可以安装命令行的组件:

```sh
opkg install samba36-client
```

也可以安装图形的界面, 这里推荐这一个:

```sh
opkg install luci-app-samba
```

安装好之后, 打开网页去openwrt的网页接口, 应该多了一个`services`接口, 点开里面有`Network Shares`, 配置如下:

![](/img/programming/share.jpg)

其中Name可以自己取名字, Path指的是要分享的路径。这里直接设置成`/mnt`下全部内容即可。其余选项按照如图所示, 然后保存即可。如果有问题, 也可以参考[官方文档](https://openwrt.org/docs/guide-user/services/nas/usb-storage-samba-webinterface)。

## 3. 支持

配置好之后, 就可以用ipad看视频了, 推荐使用PlayerXtreme, 可以支持网络播放。里面网络设备里应该会自动发现。此外, iPad可以使用Terminus来作为SSH连接路由器, 方便好用。

### 3.1 卸载

用完之后, 建议取消挂载, 但取消挂载可能显示设备正忙, 使用命令:

```sh
umount -l /mnt/sda1/
```

### 3.2 速度

由于设备简单, 速度也不能达到很快, USB设备速度, 网络转换速度, Samba速度都可能称为瓶颈, 其中Samba应该是速度的瓶颈, [官方参考](https://openwrt.org/docs/techref/hardware/performance)速度大概为:

- Read: 8.2 MiB/s (stock firmware reaches about 6.5 MiB/s under similar conditions)
- Write: 3.1 MiB/s


## 4. 其他

这个是对上文部分内容的进一步解释：

##### (1) USB支持

对于[`kmod-usb-storage`](https://openwrt.org/packages/pkgdata/kmod-usb-storage)和[`kmod-usb2`](https://openwrt.org/packages/pkgdata/kmod-usb2)这两个package, 前者是USB存储的支持, 后者拓展了相关的控制接口。此外, 如果对于U盘识别还存在问题, 那么还需要安装其他接口。[这里](https://oldwiki.archive.openwrt.org/doc/howto/usb.essentials#modules_for_usb_20)给出了其他的USB接口的安装和用法, 包括目前已经用的很多的USB 3.0等(我的路由器是老机子。。。所以只装了2.0)。

各种协议:

Name|	Size|	Required|	Desciption
---| --- | --- | ---
kmod-usb-core	|74274|	yes|	Kernel support for USB.
kmod-usb-ohci	|14935|	specific|	Kernel support for USB OHCI controllers.
kmod-usb-uhci	|14897|	specific|	Kernel support for USB UHCI controllers.
kmod-usb2	|24752|	specific|	Kernel support for USB2 (EHCI) controllers.
kmod-ledtrig-usbdev	|3502|	no|	Kernel module to drive LEDs based on USB device presence/activity.
usbutils	|187087|	no|	USB devices listing utilities: lsusb, …
kmod-leds-wndr3700-usb	|2156|	no|	Kernel module for the USB LED on the Netgear WNDR3700 board only.

其中OHCI是早期USB 1.1, UHCI是USB 1.x。我们一般使用的USB 2.0是EHCI。进一步拓展的xHCI是USB最新的接口。关于这部分的介绍可以参考[wikipedia][1]

[1]: https://en.wikipedia.org/wiki/Host_controller_interface_(USB,_Firewire)#USB

##### (2) 文件系统支持

openwrt支持的常见的文件系统的package:

Package Name|	Description
---|---
kmod-fs-autofs4	| Kernel module for AutoFS4 support
kmod-fs-btrfs	| Kernel module for BTRFS support
kmod-fs-cifs	| Kernel module for CIFS support
kmod-fs-configfs	| Kernel module for configfs support
kmod-fs-cramfs	| Kernel module for cramfs support
kmod-fs-exportfs	| Kernel module for exportfs. Needed for some other modules.
kmod-fs-ext4	| Kernel module for EXT4 filesystem support
kmod-fs-f2fs	| Kernel module for F2FS filesystem support
kmod-fs-fscache	| General filesystem local cache manager
kmod-fs-hfs	| Kernel module for HFS filesystem support
kmod-fs-hfsplus	| Kernel module for HFS+ filesystem support
kmod-fs-isofs	| Kernel module for ISO9660 filesystem support
kmod-fs-jfs	| Kernel module for JFS support
kmod-fs-minix	| Kernel module for Minix filesystem support
kmod-fs-msdos	| Kernel module for MSDOS filesystem support
kmod-fs-nfs	| Kernel module for NFS client support
kmod-fs-nfs-common |	Common NFS filesystem modules
kmod-fs-nfs-common-rpcsec	| Kernel modules for NFS Secure RPC
kmod-fs-nfs-v3	| Kernel module for NFS v3 client support
kmod-fs-nfs-v4	| Kernel module for NFS v4 support
kmod-fs-nfsd	| Kernel module for NFS Kernel server support
kmod-fs-ntfs	| Kernel module for NTFS filesystem support
kmod-fs-reiserfs	| Kernel module for ReiserFS support
kmod-fs-squashfs	| Kernel module for SquashFS 4.0 support
kmod-fs-udf	| Kernel module for UDF filesystem support
kmod-fs-vfat	| Kernel module for VFAT filesystem support
kmod-fs-xfs	| Kernel module for XFS support

以上格式是openwrt完全支持的, 就是可以进行磁盘操作的, 比如分区, 格式化, 检查等。但是关于安装`exFAT`格式的驱动, 需要[自己编译](https://openwrt.org/docs/guide-user/additional-software/beginners-build-guide)。具体怎么操作不是很懂。。。。。总之就是很麻烦。

此外, openwrt作为一个Linux系统, 有两种格式支持的最好, 一种是`ext4`, 即Linux文件系统, 对于硬件驱动支持的最好, 另一种即`F2FS`, 对于闪存设备支持的最好。

而像是`NTFS`这种windows下的文件系统, 支持就不那么好了。关于各种文件系统的支持及简介, 参考[官方文档](https://openwrt.org/docs/guide-user/storage/filesystems-and-partitions)

实验了下,NTFS也可以挂载,速度确实慢一些. 要安装`ntfs-3g`:

```
opkg install ntfs-3g
```

挂载命令:
```
sudo mount -t ntfs-3g -o uid=$(id -u) /dev/sdb1 /mnt/
```

## 5. 参考

主要参考内容都在文中连接给出了, 此外配置中参考了[这篇文章](https://spotlightcybersecurity.com/openwrt-usb-storage.html)
