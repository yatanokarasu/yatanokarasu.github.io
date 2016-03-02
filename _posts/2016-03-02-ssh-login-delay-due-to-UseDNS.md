---
layout:         post
title:          SSH login delay due to UseDNS
description:    "SSHログイン時に時間がかかる場合、UseDNSが問題かもしれません。"
keywords:       [SSH, remote login, UseDNS]
tags:           [SSH]
---

If we face SSH login delay, we should review `UseDNS` parameter in `/etc/ssh/sshd_config`.

When `UseDNS yes`, a reverse DNS lookup for SSH client IP address is enable.
When the server can not lookup hostname of the client, it is wasteful time.
The client can not login for the lookup time out.

`UseDNS` is default `yes`, so it should be set `UseDNS no` right away after construction of the server.

- - -

もしネットワーク負荷が高いわけでもないのにSSHログインに時間がかかる場合、`/etc/ssh/sshd_config`の`UseDNS`の値を確認したほうが良い。

`UseDNS yes`に設定されている場合、SSHクライアントのIPアドレスを逆引きする。
これにより、DNSの利用ができない内部ネットワークにあるサーバ等の場合、解決もできないSSHクライアントのIPアドレスを頑張って逆引きを行い、
タイムアウトすることでやっとSSHログインが行われる。

デフォルトで`UseDNS yes`が設定されているので、サーバ構築後は忘れずに`UseDNS no`に設定することが望ましい。
