---
layout:         post
title:          "Usage firewalld released from CentOS 7"
description:    "The iptables command is deprecated from RHEL7/CentOS 7. Alternatively, we should use firewalld."
keywords:       [CentOS7, firewalld]
categories:     [Linux, Infrastructure]
tags:           [CentOS7, Security]
---

RHEL7/CentOS 7から、ファイアウォールの`iptables`ではなく`firewalld`を使うようになりました。
内部的には`iptables`を使っているようなのですが、両方を同時に使用することはできません。
これまで通り`iptables`を使用することもできますが、`firewalld`も使えるようになっておきたいと思います。

<!-- more -->

# Zone

`firewalld`は以下のような設計になっています。

*   インターフェース(NIC)単位でファイアウォールを設定する
*   インターフェースに対してゾーンという概念を設定する
*   ゾーン毎に`SSH`や`HTTP`など提供するサービスの許可/拒否を設定する

ゾーンは以下のものがあります。デフォルトは`public`ゾーンです。

| Zone                 | 定義 |
|:---------------------|:----|
| drop | 着信パケットは全て遮断され、返信もされません。送信のみ可能です。 __変更は不可です__ |
| block | 全ての着信パケットは`icmp-host-prohibited`で拒否します。システムによって作成されたコネクション接続のみが可能です。 __変更は不可です__ |
| public | __インストール時のデフォルト設定です__　公開エリア用です。ネットワーク上の他のコンピュータを信頼しません。選択された着信接続のみ有効です。デフォルトでは`ssh`と`dhcpv6-client`が有効になっています。 |
| external             | ルータ用にマスカレードを有効にした外部ネットワーク向けです。ネットワーク上の他のコンピュータを信頼しません。選択された着信接続のみ有効です。デフォルトでは`ssh`とIPマスカレードが有効になっています |
| dmz                  | 公開アクセスが可能ではあるものの、内部ネットワークへのアクセスには制限があるDMZにあるコンピュータ用。選択された着信接続のみ有効です。デフォルトでは`ssh`のみ有効になっています。 |
| work                 | 作業エリア用です。ネットワーク上の他のコンピュータをほぼ信頼します。選択された着信接続のみ有効です。デフォルトでは`ssh`、`ipp-client`、`dhcpv6-client`が有効になっています。 |
| home                 | ホームエリア用です。ネットワーク上の他のコンピュータをほぼ信頼します。選択された着信接続のみ有効です。デフォルトでは`ssh`、`ipp-client`、`dhcpv6-client`、`mdns`、`samba-client`が有効になっています。 |
| internal             | 内部ネットワーク用です。ネットワーク上の他のコンピュータをほぼ信頼します。選択された着信接続のみ有効です。デフォルトでは`ssh`、`ipp-client`、`dhcpv6-client`、`mdns`、`samba-client`が有効になっています。 |
| trusted  | すべてのネットワーク接続が許可されます。 __変更は不可です__ |
{: rules="rows"}


# Usage

さっそく使ってみます。インストールされていなければ、下記コマンドでインストールします。

~~~
# yum -y install firewalld
# systemctl start firewalld
~~~


`firewalld`を操作するCLIは`firewall-cmd`です。まずは起動状態を確認します。`running`と表示されれば稼働しています。

~~~
# firewall-cmd --state
running
~~~

`--list-all`オプションでデフォルトゾーンを確認します。`public`ゾーンがデフォルトになっています。`interfaces`に何もインターフェース名が表示されていないので、
現時点では`public`ゾーンで制御するインターフェースが設定されていないことを示しています。

~~~
# firewall-cmd --list-all
public (default)
  interfaces:
  sources:
  services: dhcpv6-client ssh
  ports:
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:
~~~

デフォルトゾーンがどこかを確認するだけであれば、`--get-default-zone`オプションが使用できます。

~~~
# firewall-cmd --get-default-zone
public
~~~

他のゾーンの状況はどうでしょうか。`--list-all-zones`オプションで、全てのゾーンの状況を確認できます。

~~~
# firewall-cmd --list-all-zones
block
  interfaces:
  sources:
  services:
  ports:
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:

dmz
  interfaces:
  sources:
  services: ssh
  ports:
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:

drop
  interfaces:
  sources:
  services:
  ports:
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:

external
  interfaces:
  sources:
  services: ssh
  ports:
  masquerade: yes
  forward-ports:
  icmp-blocks:
  rich rules:

home
  interfaces:
  sources:
  services: dhcpv6-client ipp-client mdns samba-client ssh
  ports:
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:

internal
  interfaces:
  sources:
  services: dhcpv6-client ipp-client mdns samba-client ssh
  ports:
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:

public (default)
  interfaces:
  sources:
  services: dhcpv6-client ssh
  ports:
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:

trusted
  interfaces:
  sources:
  services:
  ports:
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:

work
  interfaces:
  sources:
  services: dhcpv6-client ipp-client ssh
  ports:
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:
~~~


ゾーン名の一覧を表示するだけであれば`--get-zones`が使用できます。

~~~
# firewall-cmd --get-zones
block dmz drop external home internal public trusted work
~~~

ここで、`--list-all-zone`の結果を見る限り、どのゾーンにもインターフェースが設定されていません。
しかし、ゾーンに全くいんたーふぇーすが設定されていないからと言って、全くファイアウォールが有効になっていないかというと、そうでもありません。
`iptables`コマンドでルールを確認すると、`public`ゾーンにならって`SSH`の`22`の`SYN`パケットのみ有効になっています。

~~~
# iptables -L -n --line
  :
Chain IN_public_allow (1 references)
num  target     prot opt source               destination
1    ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:22 ctstate NEW
  :
~~~

これは`systemctl stop firewalld`で`firewalld`を止めたとしても残ったままです。
完全に開放したい場合は`iptables -F`で設定を削除する必要があります。これらの設定は`systemctl start firewalld`で次回起動した際に、
設定ファイルから再現されます。設定ファイルは、もちろん`iptables`のものではなく、`firewalld`が管理するファイル群です。

# Activate Zone

インターフェースが設定されているゾーンは`active`ゾーンとして扱われます。
インターフェースを追加するには`--add-interface`オプションで行います。ここでのインターフェースは`eth0@if7`という名前です。

~~~
# firewall-cmd --add-interface=eth0@if7
success
~~~

どこのゾーンに追加されたかというと、デフォルトゾーンの`public`に追加されています。また、`public`ゾーンが`active`になっていることも確認できます。

~~~
# firewall-cmd --list-all
public (default, active)
  interfaces: eth0@if7
  sources:
  services: dhcpv6-client ssh
  ports:
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:
~~~

現在`active`なゾーンを確認するには`--get-active-zone`オプションを指定します。
ゾーン毎に設定されているインターフェースの一覧も確認可能です。

~~~
# firewall-cmd --get-active-zone
public
  interfaces: eth0@if7
~~~



デフォルトゾーンを変更したい場合は`--set-default-zone`を指定します。試しに`work`ゾーンに変更してみます。

~~~
# firewall-cmd --set-default-zone=work
success

# firewall-cmd --list-all
work (default, active)
  interfaces: eth0@if7
  sources:
  services: dhcpv6-client ipp-client ssh
  ports:
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:
~~~

先ほど`public`にゾーンに追加したインターフェースまで`work`ゾーンに移ってきています。
これはデフォルトゾーンに設定されているインターフェースは変更先のゾーンに移ってくるためです。
`--zone`オプションを指定することで、特定のゾーンの情報を確認できますので、
`public`ゾーンを指定すると、先ほど設定したインターフェースがないことが分かります。

~~~
# firewall-cmd --zone=public --list-all
public
  interfaces:
  sources:
  services: dhcpv6-client ssh
  ports:
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:
~~~

もし、インターフェースだけゾーン変更する場合は`--change-interface`オプションで行います。

~~~
# firewall-cmd --zone=public --change-interface=eth0@if7
success
public (active)
  interfaces: eth0@if7
  sources:
  services: dhcpv6-client ssh
  ports:
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:
~~~

以降の説明は`public`ゾーンがデフォルトとして進めますので、`firewall-cmd --set-default-zone=public`で元に戻しておきます。


# Accept Services

では、いよいよサービスを設定していきます。
ここでは`httpd`をインストールして、`80/tcp`でウェブアクセスできるようにしてみます。
`httpd`は`yum -y install httpd && systemctl start httpd.service`でインストールおよび起動しておきます。

サービスを追加するには`--add-service`オプションを指定します。ここではデフォルトの`public`ゾーンに追加しています。

~~~
# firewall-cmd --add-service=http
success
~~~

ゾーンに設定されているサービスの一覧は`--list-services`オプションで取得できます。`http`が追加されていることが確認できます。

~~~
# firewall-cmd --list-services
dhcpv6-client http ssh
~~~

指定できるサービス名は`--get-services`オプションで取得できます。

~~~
# firewall-cmd --get-services
RH-Satellite-6 amanda-client bacula bacula-client dhcp dhcpv6 dhcpv6-client dns freeipa-ldap freeipa-ldaps freeipa-replication ftp high-availability http https imaps ipp ipp-client ipsec iscsi-target kerberos kpasswd ldap ldaps libvirt libvirt-tls mdns mountd ms-wbt mysql nfs ntp openvpn pmcd pmproxy pmwebapi pmwebapis pop3s postgresql proxy-dhcp radius rpc-bind rsyncd samba samba-client smtp ssh telnet tftp tftp-client transmission-client vdsm vnc-server wbem-https
~~~

`iptables`でも確認してみます。`public`ゾーンとして`80/tcp`が許可されていることが確認できます。

~~~
# iptables -L -n --line 
   :
Chain IN_public_allow (1 references)
num  target     prot opt source               destination
1    ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:22 ctstate NEW
2    ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:80 ctstate NEW
   :
~~~

しかし、この設定は`firewalld`を再起動すると消えてなくなります。

~~~
# systemctl restart firewalld

# firewall-cmd --list-services
dhcpv6-client ssh

# ipables -L -n --line
   :
Chain IN_public_allow (1 references)
num  target     prot opt source               destination
1    ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:22 ctstate NEW
   :
~~~

設定を永続化するには`--permanent`オプションを指定します。

~~~
# firewall-cmd --permanent --add-service=http
success
~~~

しかし、現在の設定を確認しても、追加したはずの`http`が設定されていません。

~~~
# firewall-cmd --list-services
dhcpv6-client ssh

# iptables -L -n --line
   :
Chain IN_public_allow (1 references)
num  target     prot opt source               destination
1    ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:22 ctstate NEW
   :
~~~

これは、`--permanent`オプションは設定ファイルへの永続化を行いますが、現在稼働中の設定には反映されないためです。
設定ファイルに永続化した情報を反映するには、`--reload`オプションで再読み込みする必要があります。
ちなみに、`iptables`と異なり、設定中にパケットがロストすることはありません。

~~~
# firewall-cmd --reload
success

# firewall-cmd --list-service
dhcpv6-client http ssh
~~~


ファイアウォールの設定として、もちろんポート指定なども可能です。
いったん、設定した`http`の設定を削除します。削除は`--remove-service`オプションを使用します。

~~~
#　firewall-cmd --remove-service=http --permanent
success

# firewall-cmd --reload
success
~~~


なんとなく予想ができるかもしれませんが、ポートを指定するには`--add-port`オプションを指定します。

~~~
# firewall-cmd --add-port=80/tcp
success

# iptables -L -n --line
   :
Chain IN_public_allow (1 references)
num  target     prot opt source               destination
1    ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:22 ctstate NEW
2    ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:80 ctstate NEW
   :
~~~


# Configuration files

`firewalld`自体の設定ファイルは`/etc/firewalld/firewalld.conf`にあります。
`DefaultZone`で`public`が指定されていることが確認できます。

<span class="balloon">/etc/firewalld/firewalld.conf</span>

~~~ bash
# firewalld config file

# default zone
# The default zone used if an empty zone string is used.
# Default: public
DefaultZone=public

# Minimal mark
# Marks up to this minimum are free for use for example in the direct
# interface. If more free marks are needed, increase the minimum
# Default: 100
MinimalMark=100

# Clean up on exit
# If set to no or false the firewall configuration will not get cleaned up
# on exit or stop of firewalld
# Default: yes
CleanupOnExit=yes

# Lockdown
# If set to enabled, firewall changes with the D-Bus interface will be limited
# to applications that are listed in the lockdown whitelist.
# The lockdown whitelist file is lockdown-whitelist.xml
# Default: no
Lockdown=no

# IPv6_rpfilter
# Performs a reverse path filter test on a packet for IPv6. If a reply to the
# packet would be sent via the same interface that the packet arrived on, the
# packet will match and be accepted, otherwise dropped.
# The rp_filter for IPv4 is controlled using sysctl.
# Default: yes
IPv6_rpfilter=yes
~~~

ゾーン毎の設定ファイルは`/etc/firewalld/zones`配下に格納されたXMLファイルです。
下記は`public`ゾーンの設定ファイルです。

<span class="balloon">/etc/firewalld/zones/public.xml</span>

~~~ xml
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.</description>
  <service name="dhcpv6-client"/>
  <service name="ssh"/>
</zone>
~~~

`service`要素で、`dhcpv6-client`と`ssh`だけ設定されていることが確認できます。
ここで、`firewall-cmd --add-service=http --permanent`を投入後に再度確認してみます。

~~~
# firewall-cmd --add-service=http --permanent
success

# cat /etc/firewalld/zones/public.xml
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.</description>
  <service name="dhcpv6-client"/>
  <service name="http"/>
  <service name="ssh"/>
</zone>
~~~

設定ファイルに`http`サービスが追加されて、永続化されていることが確認できます。

ポート指定の場合も同様です。

~~~
# firewall-cmd --add-port=8080/tcp --permanent
success

# cat /etc/firewalld/zones/public.xml
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.</description>
  <service name="dhcpv6-client"/>
  <service name="http"/>
  <service name="ssh"/>
  <port protocol="tcp" port="8080"/>
</zone>
~~~

`port`要素に`8080/tcp`が追加されていることが確認できます。


- - -


以上が、大まかな`firewalld`の操作方法です。
もちろん、フォワードやマスカレードの設定などもできまし、`iptables`以上に柔軟に設定することが可能です。

今日はここまで。
