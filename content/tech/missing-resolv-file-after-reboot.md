+++
Description = "resolv.conf配置文件信息 重启 丢失"
Tags = ["resolv.conf","配置信息","重启","丢失"]
date = "2012-05-08T20:21:16+08:00"
ds_title = "resolv.conf 配置信息重启后丢失解决方法"
title = "resolv.conf 配置信息重启后丢失解决方法"

+++
&nbsp;

我要配置DNS，修改/etc/resolv.conf，修改后重启服务 service network restart ，修改/etc/resolv.conf的信息丢失，请大家看看。

修改前的配置
```C
# No nameservers found; try putting DNS servers into your
# ifcfg files in /etc/sysconfig/network-scripts like so:
#
# DNS1=xxx.xxx.xxx.xxx
# DNS2=xxx.xxx.xxx.xxx
# DOMAIN=lab.foo.com bar.foo.com
```
&nbsp;

<wbr></wbr>

&nbsp;

解决方法在ifcfg-eth0 直接加入DNS1=xxx.xxx.xxx.xxx，再service network restart

&nbsp;
```C
DEVICE="eth0"
BOOTPROTO="static"
HWADDR="00:0C:29:B5:E4:65"
NM_CONTROLLED="yes"
ONBOOT="yes"
IPADDR=192.168.128.133
NETMASK=255.255.255.0
GATEWAY=192.168.128.1
DNS1=222.46.120.6
```
&nbsp;

再来查看resolv.conf 多了一行 namerserver xxx.xxx.xxx.xx
# Generated by NetworkManager
nameserver 222.46.120.6

&nbsp;

<wbr></wbr>

&nbsp;

<wbr></wbr>

&nbsp;

还有方法二：<wbr></wbr>

终于找到一篇文章解决了我的问题：http://tech.techweb.com.cn/archiver/tid-380658.html
文章内容：
vim /etc/resolvconf/resolv.conf.d/head 文件
显示与resolv.conf相同的内容：
# Dynamic resolv.conf(5) file for glibc resolver(3) generated by resolvconf(8)
# DO NOT EDIT THIS FILE BY HAND -- YOUR CHANGES WILL BE OVERWRITTEN)
在最后键入nameserver 202.102.152.3
保存退出，
resolvconf -u
此时就可以正常上网了，重启后不用在重新设置DNS了。

&nbsp;

&nbsp;

实测：方法1很有效~
