---
title: "关于SSH攻击以及安全防护的小科普"
date: 2020-07-29T16:57:00+08:00
tags: ["计算机安全"]
categories: ["计算机"]
---

# 记一次服务器被攻击

> 人在家中坐，锅从天上来。

## 且说

说我作为一个遵纪守法的好青年，在~~毕业~~失业后就乖乖在家好好学习，天天向上。

没想到，天有不测风云，就在昨天，我突然收到了服务器商的邮件警告：~~FBI WARNING~~（大误）。说我的服务器涉嫌违法，已经被关停。迷惑三连后，我发了工单询问，等了好久后，服务商说：原来他们搞错IP了....

我的内心：@#￥#@￥……&#……￥*&@…#￥&*

我只能无可奈何地把服务器上原来的服务重新部署....

但没想到今天早上又出现新的bug：SSH正常，但一些特定流量始终走不出去，报错是无法解析域名。经过一番~~周密~~的debug后，我觉得有可能是服务商更新了什么防火墙策略造成的。正准备发工单问，结果打开网站赫然看到：**由于硬件故障，机房的外网通讯受到影响。**

最后看到公告，原来是运营商GTT/001899468核心业务卡故障，P0级别重大事故。。。。。。。

另外最近发现服务器有时候SSH连接特别慢。因为有~~一堵看不见的墙~~，我觉得这种事情很正常，因为会存在随机tcp丢包的现象。特别我经常会把一些数据走SSH隧道，这样的大流量行为被丢包也很正常。

但是在运营商那边修复后，我SSH完全连不上了.....我很懵逼，难道我的服务器被封了？于是我`ping`了一下，发现不仅没被封而且稳如老狗。

这就很奇怪了，但是听说在某些情况ping（ICMP协议）不会被阻断，但TCP会被阻断。于是我去找了一款[tcping工具](https://github.com/cloverstd/tcping)来测试tcping连接，然后发现tcping也稳如老狗.....

![](/img/ssh_attack/Snipaste_2020-07-29_17-22-42.png)

这就很迷惑了，于是我看了下SSH连接的详细信息，发现卡在了`SSH2_MSG_KEXINIT`这一步，大概就是服务器那边不给响应。就很奇怪，服务器工作正常，为什么不响应我的连接请求呢？~~难道是因为哥长得帅？~~

这时候搜到一个帖子说，可能是遭遇到了SSH攻击，SSHD会丢掉一些连接。

我恍然大悟，赶快重启了机子，顺利连上了SSH，然后一看log，我和小伙伴们都惊呆了，发现居然有三万多次登录失败的记录：

```
root@ubuntu:/var/log# grep -o "Failed password" auth.log | uniq -c
  33855 Failed password
```

看一下前面的日志居然更多

```
root@ubuntu:/var/log# grep -o "Failed password" auth.log.1 | uniq -c
  74975 Failed password
```

原来这几天SSH一直被大量访问，怪不得偶尔这么慢。

幸好我的密码不是太简单（其实也很简单，但是能保证不会被最基本的字典攻破）。出于安全起见，我赶快把root登录关闭（修改`/etc/ssh/sshd_config`文件）：

```
PermitRootLogin no
```

把root登录关闭的话，要成功登录SSH就必须保证用户名和密码都正确才行，这样就不那么容易被破解了，而且登录后进行高权限操作也还得再破解root的密码。

修改后重启sshd：

```
systemctl restart sshd
```

> 注意在操作sshd的过程中要小心，万一弄错了配置，导致ssh连不上服务器就麻烦了

但是光做这个工作还不够，如何防范SSH攻击呢？有几种思路：

- IP黑名单，限制攻击我服务器的IP
- SSH指定用户：仅允许指定的用户登录
- IP白名单，仅仅允许指定的IP登录
- PAM（Pluggable Authentication Modules）：引入额外的身份验证。例如对同一个登录用户进行时间锁定，登录失败超过几次后锁定一定时间。手机和电脑的密码都采用了这样的保护方式。

但哪种方式更适合我呢？继续研究一下log：


首先看看攻击我的IP有多少：

```
root@ubuntu:/var/log# grep "Failed password" auth.log | grep -E -o "([0-9]{1,3}[\.]){3}[0-9]{1,3}" | wc -l
33861
```

喵喵喵？所以居然还是分布式攻击，几乎完全就是一个IP对应一次攻击.....我这小破服务器性能差，又没什么资料，居然得到如此殊誉，简直震惊！！！

那顺便看一下都是怎么登录的：

```
root@ubuntu:/var/log# grep "Failed password" auth.log | perl -e 'while($_=<>){ /for(.*?) from/; print "$1\n";}' | uniq 

......

 invalid user chenjiayun
 invalid user liaojh
 invalid user Tlhua
 invalid user rd5
 invalid user root
 invalid user zhangrong
 invalid user root
 invalid user xli
 invalid user root
 invalid user kumagai
 invalid user kk
 invalid user gsqc
 invalid user bluewing
 invalid user tangqw
 invalid user root
 invalid user SuZhixin
 invalid user yuri
 invalid user zhongchen
 invalid user root
 invalid user annakaplan
 invalid user root
 invalid user hanlj
 invalid user ishihara
 invalid user lzhou
 invalid user root
 invalid user magfield
 invalid user liulei
 invalid user zhaozhehuan
 invalid user zhanghongwei
 invalid user root
 invalid user zexin
 invalid user root
 invalid user minjie
 invalid user root
 invalid user alias
 invalid user zzz
 invalid user colfax
 invalid user yangdeyue
 invalid user root
 invalid user fzs
 invalid user root
 invalid user peihongtao
 invalid user root
 invalid user maggie
 invalid user zhenyuan
 invalid user jiangtao
 invalid user ashish
 invalid user aaron
 invalid user root
.....
```

看到这里大概明白了，这应该是**nmap配合社工库做的自动脚本**。但我这小破服务器也有点夸张吧，这么多访问都影响到我的登录了。

所以简单来说，这就相当于一种DDOS攻击，即大量的用户请求导致服务卡死，无法满足正常的请求。所以要保证不影响我自己的操作，需要防止大量的请求到sshd上，解法就呼之欲出了：用ufw防火墙建立白名单好了。

ufw操作使用很简单：

安装ufw：

```
apt-get install ufw
```

开启ufw：

```
ufw enable
```

编辑配置文件，如果你的机器自带IPv6地址，确保“IPV6=yes”这个值生效，：

设置基本的入站出战：

```sh
# 入站禁止所有端口访问，后面会预留必要的端口：
ufw default deny incoming
# 出站允许服务器访问所有端口：
ufw default allow outgoing
```

然后按照自己的需求添加白名单：

```
ufw allow from XXX.XXX.XXX.XXX
.....
```

弄好后，重载ufw即可：

```
ufw reload
```

> 注意这里操作也要小心，务必保证正确并且确定好自己的IP，不然到时候把自己阻隔在外面就麻烦了。。。

### SSHD错误的配置

一些常见的错误的设置：

- `MaxAuthTries`：这个看起来是指最多尝试次数，超过次数socket就会被关闭。但是这并没有任何多余的限制，关闭的socket甚至不需要更换IP就可以马上重新建立，因此根本没有办法防止暴力破解。

### 附：关于ufw的其它一些操作


- 查看ufw的运行情况：

    ```
    ufw status verbose
    ```

- 查看ufw的log，首先开启ufw的log记录：

    ```
    ufw logging on
    ```

    然后搜索看一下，就可以发现大量被阻隔的请求：

    ```
    grep -i ufw /var/log/syslog
    ```

- ufw添加/拒绝端口

    ```
    # 允许80端口（包括tcp、udp）
    ufw allow 80
    # 允许80端口的tcp
    ufw allow 80/tcp
    # 允许80端口的udp
    ufw allow 80/udp
    ```

    拒绝端口把`allow`换成`deny`即可。

- ufw删除规则

    ```
    # 列出规则和编号
    ufw status numbered
    # 按照编号删除，例如删除第二条规则
    ufw delete 2
    ```

- ufw允许特定的IP和端口

    ```
    sudo ufw allow from <target> to <destination> port <port number>
    ```

    例如允许`192.168.0.1`用任何协议访问22端口：

    ```
    ufw allow 192.168.0.1 to any port 22
    ```

    如果需要协议，就在后面增加协议：

    ```
    sudo ufw allow from <target> to <destination> port <port number> proto <protocol name>
    ```

    例如：

    ```
    ufw allow 192.168.0.1 to any port 22 proto tcp
    ```


### 参考

- [Linux应急响应（一）：SSH暴力破解](https://www.cnblogs.com/xiaozi/p/9561423.html)
- [系统安全之SSH入侵的检测与响应](https://www.freebuf.com/articles/system/194775.html)
- [Allow Or Deny SSH Access To A Particular User Or Group In Linux](https://ostechnix.com/allow-deny-ssh-access-particular-user-group-linux/)
- [UFW Essentials: Common Firewall Rules and Commands](https://www.digitalocean.com/community/tutorials/ufw-essentials-common-firewall-rules-and-commands#:~:text=Allow%20Incoming%20SSH%20from%20Specific%20IP%20Address%20or%20Subnet&text=For%20example%2C%20if%20you%20want,24%20to%20any%20port%2022)
- [How to List and Delete UFW Firewall Rules](https://linuxize.com/post/how-to-list-and-delete-ufw-firewall-rules/)
- [简明教程｜Linux中UFW的使用](https://zhuanlan.zhihu.com/p/98880088)
- [Federico Kereki：保护SSH的三把锁](https://www.ibm.com/developerworks/cn/aix/library/au-sshlocks/index.html)
- [SSH会话劫持实现端口转发](https://www.freebuf.com/articles/network/61579.html)
- [RegEx: Find IP Addresses in a File Using Grep](https://www.shellhacks.com/regex-find-ip-addresses-file-grep/)
- [How to Set Up a Firewall with UFW on Ubuntu 18.04](https://linuxize.com/post/how-to-setup-a-firewall-with-ufw-on-ubuntu-18-04/)