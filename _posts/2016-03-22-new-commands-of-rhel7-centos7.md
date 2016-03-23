---
layout:         post
title:          "New commands of RHEL7/CentOS7 for network configuration"
description:    "The 'net-tools' package is deprecated from RHEL7/CentOS, we should use the 'iproute' package."
keywords:       [CentOS7, iproute2]
categories:     [Linux]
tags:           [CentOS7, iproute2]
---

RHEL7/CentOS7から、最小構成でインストールする`ipconfig`や`netstat`などのネットワーク関連のコマンドが入っていません。
ずっと慣れ親しんだコマンドが使えないのはかなり困るのですが、下記コマンドでインストールすることができます。

~~~
# yum -y install net-tools
~~~

しかし、`net-tools`パッケージはdeprecatedになっているようです。無くなっても困らないように、新しいコマンドを使いこなしたいですね。

<!-- more -->

# New commands for RHEL7/CentOS

`net-tools`パッケージのコマンドの代わりになる`iproute`パッケージの新しいは以下です。

| net-tools             | iproute                       | means                         |
|:----------------------|:------------------------------|:------------------------------|
| `ifconfig`            | `ip a(addr)`, `ip l(link)`    | Show address and device list  |
| `netstat`             | `ss`                          | Show socket list              |
| `netstat -i`          | `ip -s l(link)`               | Show network interface table  |
| `netstat -l`          | `ss -l`                       | Show `LISTEN` socket list     |
| `netstat -r`, `route` | `ip r(route)`                 | Show routing table            |
| `arp -n`              | `ip n(neighbor)`              | Show ARP table                |
{: rules="groups"}


## "`ifconfig`" vs. "`ip a(addr)`"

__In the case of__ `ifconfig`

{% highlight raw %}
# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.2  netmask 255.255.0.0  broadcast 0.0.0.0
        inet6 fe80::42:acff:fe11:2  prefixlen 64  scopeid 0x20<link>
        ether 02:42:ac:11:00:02  txqueuelen 0  (Ethernet)
        RX packets 4451  bytes 25737167 (24.5 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 4087  bytes 222342 (217.1 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 0  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
{% endhighlight %}

__In the case of__ `ip a(addr)`

{% highlight raw %}
# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
13: eth0@if14: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:acff:fe11:2/64 scope link
       valid_lft forever preferred_lft foreve
{% endhighlight %}

`route`コマンドで操作していたルーティングテーブルも、`ip r(route)`コマンドで操作になります。


## "`netstat`" vs "`ss`"

`ss`コマンドには、`-a`オプションで全てのソケットを表示した場合、UDPソケットまでTCPソケットとして表示されるバグがあるようです。詳細は[コチラ](http://d.hatena.ne.jp/ozuma/20140915/1410774381)
いつか修正されるでしょう。

コマンドの使用感については、`netstat`で使用していたオプションは、基本的にそのまま使用できます。


今度は`firewalld`とかも調べてみよう。
