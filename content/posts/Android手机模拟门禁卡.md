---
title: "Android手机模拟门禁卡"
date: 2019-06-26
draft: false
tags: ["硬件"]
categories: ["计算机"]
---

> 注意! 以下内容仅用作技术交流与学习, 如他人使用本资料引发法律问题一改与作者无关! 

## 先说说卡

目前常用的卡按照读取方式来看，可以分为磁条卡和IC卡两种。

- [磁条卡](https://en.wikipedia.org/wiki/Magnetic_stripe_card)，磁条卡的历史比较早，和磁性存储介质发展的时间差不多，最早由IBM公司的员工[Forrest Parry](https://en.wikipedia.org/wiki/Forrest_Parry)提出。广泛用在储蓄卡和信用卡上。记得一些超市的购物卡也是这种卡。
- IC卡，也叫智慧卡(smart card)，芯片卡(chip card)，集成电路卡(integrated circuit card)等。也是目前最常用最广泛的一种卡。这里就重点讲讲IC卡。

## IC卡

IC卡出现比较晚，出现在上世纪七八十年代。最早出现在法国，而大家对IC卡最广泛的印象应该都源于SIM卡，就是手机卡了。

IC卡有两种通讯方式，一种是接触式的，一种是非接触式的。接触式就是通过卡上的芯片进行通信，非接触式则是通过[RFID](https://en.wikipedia.org/wiki/Radio-frequency_identification)(射频识别)进行近场通信。在手机上热门的[NFC](https://en.wikipedia.org/wiki/Near-field_communication)技术就是从RFID上发展而来，相比RFID来讲，NFC的标准比较统一，而RFID则有多个标准，不便于推广使用。但RFID距离远，最远可达数米，应用在停车场等。

除了通讯方式, 为了实现通信的加密性. 不同的国家, 机构, 公司等也推出了不同的IC卡加密方法, 并且基于不同的加密方法和应用场合，产生了不同的卡片称呼。比如:

- [M1卡](https://en.wikipedia.org/wiki/MIFARE), 全称为MIFARE, 由飞利浦旗下[NXP公司](https://en.wikipedia.org/wiki/NXP_Semiconductors)研制, 采用M1算法, 是应用最广泛的IC卡算法. 但是在2008年被破解, 一度引发IC卡标准的混乱. 
- TM卡, Touch Memory, 由美国DALLAS公司研发。

例如下图是我的北京公交卡和上海公交卡，以及它们的信息, 可以看到, 卡的技术标准就源于NXP公司：

![](/img/Android手机模拟门禁卡/public_transpotation_card.jpg)

![](/img/Android手机模拟门禁卡/card_info.jpg)

> 扫卡的时候才发现北京的那张交通卡居然已经过期了....得去趟北京把里面的钱取出来才行....


## ID卡

那么常听到的ID卡又是怎么回事呢？ID卡指的是Identification Card，这种卡不做加密，仅通过卡的信息（比如卡的ID号，UID号）做验证。

像是安全性高的M1卡，则是M1卡发一个数据到设备，设备再传一个数据给M1卡确认，再进行交易或身份认证。

目前常用的门禁卡基本都是ID卡，广泛用在小区，公司门禁等处。这类卡几乎没有什么安全性，其原因在于国内的ID卡都是使用只读特性来进行身份识别验证，而几乎没有关注卡片和门禁之间的加密。这就造成了这类卡极易被复制，也带来了很大的安全隐患。

在淘宝就可以很容易地搜到这类型的卡片复制器：

![](/img/Android手机模拟门禁卡/card_dup.jpg)

随着手机NFC的普及, 利用手机来模拟ID卡也变成了可能. 下面就说说怎么用手机来模拟NFC卡. 

## 手机模拟ID卡

首先要有一台可以使用NFC功能的安卓手机. 

原理是这样的, 每张卡都有卡的ID, 手机的NFC功能也有一个ID, 但这个ID是动态变化的, 可以用另一台手机来扫描手机来查看. 所以要修改手机的NFC的ID, 来匹配对应的卡ID即可. 修改的时候, 需要root. 

具体修改的方式, 可以通过更改位于`/etc/`的一个叫做`libnfc-brcm-20791b05.conf`的文件, 但浩南提供了另一个思路, 可以下载一个叫做`NFC卡模拟器`的东西实现~感觉更方便好用, 就参考了他的方法, 以下实现采用浩南兄的方法. 

> 手机最初的刷机, 安装recovery等都是浩南帮我搞的~宿舍路由器的破解也是源于浩南的指点, 在这里向他表示感谢! 他是真正的老司机 XD


### 1. 安装TWRP recovery工具

TWRP recovery是在XDA论坛上出现的一款第三方recovery工具, 比安卓手机自带的recovery工具更好用, 支持更多的功能. 在刷机, root等都少不了. 

具体不同手机安装方式不同, 官方的安装方式参考: [How to Install TWRP](https://www.xda-developers.com/how-to-install-twrp/)

对应具体个人的手机, 可以以**手机型号+TWRP安装**这样的关键字来搜索. 

### 2. 使用magisk来root

手机必须要root才能更改NFC的信息, 所以要装一个root工具magisk, 安装方法的官方指南: [How to Install Magisk](https://www.xda-developers.com/how-to-install-magisk/)

这里的安装就用到了TWRP. 

### 3. 安装

装好magisk后, 安装NFC卡模拟器, 可以自己找安装包, 或者[点击我用的这个apk包](/img/Android手机模拟门禁卡/NFC Card Emulator_v4.1.0.apk)

### 4. 完成

装好后, 直接点开, 然后把要模仿的门禁卡贴在后面, 读取. 命名即可.

![](/img/Android手机模拟门禁卡/sim_card.jpg)

这里的数字就是卡的ID，模仿了这个ID就可以模仿这张卡的门禁。

亲测， 宿舍楼下门禁毫无障碍（

### 5. 后续

看到这里，肯定会有担心，这样似乎刷食堂和小卖部也毫无障碍。但并不是这样的，食堂和小卖部都是联网的，在刷卡的时候会发送交易信息，并且和结算节点的信息去核实扣款，因此这种简单的方式并不能用来刷饭卡。

虽然如此~但真实情况并非完全如此，有两个思路可以破解：

1. 在部分场合的交易是没有网络的，比如早餐车的刷卡。所以可以推测，在交易的时候，卡的额度一定写在了卡内而不仅仅是中心网络。所以可以通过这种无网络交易确认卡的额度写在哪里，进而修改额度。不过这个当然是可以查出来的，因为卡的信息和卡持有人的信息对应，而且无网络交易后的机器也会后续联网，把交易信息发送到中心结算处。因此这种破解方式适用于无记名卡。。。。比如公交卡。
2. M1卡已经证明是不安全的了，可以对卡进行暴力破解，然后使用卡进行盗刷。这种方法难度太高。。。。。


## 参考资料

除了文中自带的链接以外, 额外的看过或者引用的参考资料:

- [Smart Card Technology: Past, Present, and Future ](http://www.journal.au.edu/ijcim/2004/jan04/jicimvol12n1_article2.pdf)
- [wikipedia: smart card](https://en.wikipedia.org/wiki/Smart_card#Carte_bleue)
- [wikipedia: Radio-frequency identification](https://en.wikipedia.org/wiki/Radio-frequency_identification)
- [记一次简单的门禁卡破解尝试](https://bbs.yunxin.163.com/forum.php?mod=viewthread&tid=55)
- [Proxmark3 Easy破解门禁卡学习过程](https://lzy-wi.github.io/2018/07/26/proxmark3/)
- [比较IC卡、ID卡、M1卡、CPU卡它们之间有什么区别](https://blog.csdn.net/xiantian7/article/details/20710427)
- [如何利用Nexus 5伪造一张门禁卡](https://www.freebuf.com/news/topnews/80368.html)


## 写在最后

国内的门禁系统安全性非常低, 有的时候还是要增加一道防护. 此外, 卡丢了一定要及时挂失~万一遇到高手, 那可就惨了. 

而部分手机厂商更是以可以实现门禁作为自己售卖的噱头, 这种开放的复制卡的做法无疑更增加了安全隐患. 

以上.