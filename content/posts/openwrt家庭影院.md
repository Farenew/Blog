---
title: "OpenWrt用作家庭影院"
date: 2020-07-24
tags: ["openwrt"]
categories: ["计算机"]
---

# OpenWrt用作家庭影院

家庭影院很简单，其实只需要一个SMB或者DLNA之类的共享即可，实在不行FTP也能用。普通PC就能实现这些功能，但PC的问题在于要长期保持开机，噪声和功耗都比较大。一般的解决方案是使用NAS或者服务器，但是费用较高。

另一方面，使用路由器是一个比较好的选择，路由器一般都是不断电长期运行的，且直接提供了网络功能。目前好一点的路由器均直接提供了文件分享、云打印等功能。但这些路由器的固件都是定制的，功能固定且代码不开源，无法进一步研究。Openwrt是应用在嵌入式设备的一个Linux系统，其代码开源，功能强大。

但并不是所有设备都支持Openwrt，Openwrt对路由器的要求较高，支持Openwrt的设备可以在官网查询：https://openwrt.org/supported_devices

使用Openwrt需要有一定Linux的操作使用知识，因此这篇教程仅是一个安装说明，并不是完全小白向~

以下教程分为三部分，分别说明在Openwrt路由器上如何挂载USB设备，如何开启SMB分享和使用Openwrt安装transmission来挂PT。

## 1. OpenWrt挂载USB设备

1. SSH登录到OpenWrt上
2. 安装需要的packages：

    ```
    opkg update
    opkg install block-mount e2fsprogs kmod-usb-core kmod-usb-storage kmod-usb-storage-extras kmod-scsi-core kmod-scsi-generic kmod-usb-storage-uas kmod-usb2 kmod-usb3 
    ```

3. 根据自己需要使用的设备，安装对应的文件系统支持。

- 例如常用的FAT系统和exFAT（U盘常用的系统）：

    ```
    opkg install kmod-fs-vfat
    opkg install kmod-fs-exfat
    ```

- 例如windows用的磁盘格式NFTS：

    ```
    opkg install ntfs-3g
    ```

    注意，Opwnwrt同时还提供了`ntfs`这个包，但是这个只能读不能写，安装`ntfs-3g`更好一些。

- 例如Linux下磁盘格式ext4：

    ```
    opkg install kmod-fs-ext4
    ```

    > 其它支持的文件系统格式可以参考：https://openwrt.org/docs/guide-user/storage/filesystems-and-partitions

4. 安装`usbutils`，方便后续一些操作

    ```
    opkg install usbutils
    ```

5. 插入USB设备，查看设备是否已经插入：

    ```
    lsusb
    ```

    如果显示了设备就说明正确插入了

6. 设备应该在`/dev`下，以`sda1`开始列出来，如果只有一个设备，默认就应该是`sda1`。如果只有`sda`，那么说明设备没有插入，查看是否松动。
7. 挂载:

    ```
    mount /dev/sda1 /mnt/mount
    ```

    如果是NTFS格式的盘，可以这样挂载：

    ```
    ntfs-3g /dev/sda1 /mnt/mount -o rw,big_writes
    ```

    如果这里失败的话，查看文件系统是否支持，前面的安装有没有出问题。

    取消挂载：

    ```
    umount -l /mnt/mount/
    ```

8. 进入`/mnt/mount/`目录下查看是否已经挂载

也可以参考更详细的官方教程：

- https://openwrt.org/docs/guide-user/storage/usb-drives-quickstart
- https://openwrt.org/docs/guide-user/storage/writable_ntfs
- https://openwrt.org/docs/guide-user/storage/usb-drives

## 2. 安装SMB

安装SMB即可以SMB协议来进行流媒体传输，同样SSH进入到Openwrt：

安装Samba服务器:

```
opkg update
opkg install samba36-server
```

然后建议安装SMB的Luci界面：

```
opkg install luci-app-samba
```

安装好后，进入Openwrt的网页接口，即`192.168.1.1`，然后会出现`service`接口：

![](/img/service.png)

选择`Add`，然后设置名字和路径，名字随便取，路径按照刚才挂载的地址：

![](/img/mount.png)

注意这里的`Allow Guests`要打开。

---

然后就可以使用Ipad等设备连接，以nplayer为例：

![](/img/photo_2020-07-24_11-55-49.jpg)

这里选择SMB/CIFS即可，在其中服务器填`192.168.1.1`，其余留空，即可进入。


虽然接口是USB2.0，但是播放一些高码率的视频还是没问题的，例如播放接近10GB的影片《茜茜公主》：

![](/img/photo_2020-07-24_11-54-53.jpg)
![](/img/photo_2020-07-24_11-54-55.jpg)

唯一的问题是我的ipad很旧了，用了三年半了。。。。发热比较严重23333

## 3. 使用transmission

transmission是一款很方便的P2P下载工具，可以配置后用在PT站点的下载和上传。

OpenWrt官方提供了使用教程，参考：https://openwrt.org/docs/guide-user/services/downloading_and_filesharing/transmission

安装步骤：

```
opkg update
opkg install transmission-daemon-openssl
opkg install transmission-cli-openssl 
opkg install transmission-web
opkg install transmission-remote-openssl
```

此时transmission应该是没有启用的，我们编辑它的配置文件（在启用后无法编辑配置文件）：

这里给出我的配置示例，其中几项重要的需要配置的有：

- "download-dir"：下载的地址
- "incomplete-dir"：未完成文件的地址
- "pex-enabled"：按照PT站点的要求，一定要设置为false
- "dht-enabled"：按照PT站点的要求，一定要设置为false

```json
{
    "alt-speed-down": 50,
    "alt-speed-enabled": false,
    "alt-speed-time-begin": 540,
    "alt-speed-time-day": 127,
    "alt-speed-time-enabled": false,
    "alt-speed-time-end": 1020,
    "alt-speed-up": 50,
    "bind-address-ipv4": "0.0.0.0",
    "bind-address-ipv6": "::",
    "blocklist-enabled": false,
    "blocklist-url": "http://www.example.com/blocklist",
    "cache-size-mb": 2,
    "dht-enabled": false,
    "download-dir": "/mnt/ntfs/",
    "download-queue-enabled": false,
    "download-queue-size": 4,
    "encryption": 1,
    "idle-seeding-limit": 30,
    "idle-seeding-limit-enabled": false,
    "incomplete-dir": "/mnt/ntfs/incomplete",
    "incomplete-dir-enabled": false,
    "invalid-key": false,
    "lazy-bitfield-enabled": true,
    "lpd-enabled": false,
    "message-level": 1,
    "peer-congestion-algorithm": "",
    "peer-id-ttl-hours": 6,
    "peer-limit-global": 240,
    "peer-limit-per-torrent": 100,
    "peer-port": 51413,
    "peer-port-random-high": 65535,
    "peer-port-random-low": 49152,
    "peer-port-random-on-start": false,
    "peer-socket-tos": "default",
    "pex-enabled": false,
    "port-forwarding-enabled": false,
    "preallocation": 1,
    "prefetch-enabled": true,
    "queue-stalled-enabled": true,
    "queue-stalled-minutes": 30,
    "ratio-limit": 2,
    "ratio-limit-enabled": false,
    "rename-partial-files": true,
    "rpc-authentication-required": false,
    "rpc-bind-address": "0.0.0.0",
    "rpc-enabled": true,
    "rpc-host-whitelist": "127.0.0.1,192.168.1.*",
    "rpc-host-whitelist-enabled": false,
    "rpc-password": "{957137008a6a7931771462e8d36eae59dff8dcfdnPRlD8r9",
    "rpc-port": 9091,
    "rpc-url": "/transmission/",
    "rpc-username": "",
    "rpc-whitelist": "127.0.0.1,192.168.1.*",
    "rpc-whitelist-enabled": false,
    "scrape-paused-torrents-enabled": true,
    "script-torrent-done-enabled": false,
    "script-torrent-done-filename": "",
    "seed-queue-enabled": false,
    "seed-queue-size": 10,
    "speed-limit-down": 100,
    "speed-limit-down-enabled": false,
    "speed-limit-up": 20,
    "speed-limit-up-enabled": false,
    "start-added-torrents": true,
    "trash-original-torrent-files": false,
    "umask": 18,
    "upload-slots-per-torrent": 14,
    "utp-enabled": true,
    "watch-dir-enabled": false
}

```

然后启动：

```
uci set transmission.@transmission[0].enabled="1"
uci commit transmission
service transmission restart
```

在浏览器中打开`192.168.1.1:9091`即可操作。