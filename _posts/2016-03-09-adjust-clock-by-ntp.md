---
layout:         post
title:          "How to adjust system clock by NTI"
description:    "To adjust system clock, we can use either ntpd or ntpdate"
keywords:       [ntp, ntpdate, system clock]
categories:     [Infrastructure, Maintainance, Operation]
tags:           [Linux, NTP]
---

Linux provides two way to adjust system clock, `ntpd` and `ntpdate` command.
What is the difference of them?

- - -

Linuxではシステムクロックを調整するのに`ntpd`と`ntpdate`のいずれかが使えます。
これらの違いは何でしょうか？

<!-- more -->

# What is the difference of them

ntpd
:   This can be either server daemon to provide time or client to query to other NTP server.  
    `ntpd`コマンドは、時間を提供するサーバにもなるし、ほかのNTPサーバに時間を問い合わせることもできます。

ntpdate
:   This can only query time to NTP server.  
    `ntpdate`コマンドはNTPサーバへの時刻問い合わせだけできます。


# Behaviour to query time

There are two mode to adjust clock.  
時刻調整するために2つのモードがあります。

Step mode
:   Force adjust clock immediately. There is possibility that system clock returns to the past.  
    即座に時刻を調整します。過去に時間が戻る可能性もあります。

Slew mode
:   Adjust clock gradually. Update speed is maximum 0.05 milliseconds per second.  
    徐々に時刻を調整します。修正スピードは1秒間で最大で0.05ミリ秒です。


# Usage of two command

~~~
# ntpd -q
ntpd: time set -2.39872s
~~~

Above commans mimics that of the `ntpdate` command as follows:

~~~
# ntpdate ${NTP_SERVER}
~~~


We can register `cron` daemon script which runs above command to adjust clock periodically.
