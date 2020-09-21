---
title: "在openwrt上安装aria2"
date: 2020-08-08T11:34:40+08:00
tags: ["openwrt"]
categories: ["计算机"]
---

安装：

```
opkg install aria2 luci-app-aria2 webui-aria2
```

安装好后在luci下会出现Aria2的服务：

![](/img/openwrt/Snipaste_2020-08-08_11-36-22.png)

点进去可以看到配置，点击`WebUI-Aria2`即可进入web配置界面：

![](/img/openwrt/Snipaste_2020-08-08_11-38-08.png)