---
layout:     post
title:      How to backup MySQL binary logs
categories: [Infrastructure, Maintainance, Operation]
tags:       [MySQL, Backup/Restore]
keywords:   [MySQL, mysqlbinlog]
---


MySQLのバイナリログをバックアップする方法を考えていたところ、`mysqlbinlog`コマンドに`--read-from-remote-server`なるオプションがあることに気付いた。
その他のオプションと組み合わせて使うことで、`scp`、`ssh+tar`、`rsync`などを使用しなくてもバックアップすることが可能になる。
さらに、あたかもレプリケーションスレーブの様にふるまい、マスター側でバイナリログが更新されたら都度バックアップすることも可能になる。

<!-- more -->

オプション`--to-last-log`をつけると、指定したバイナリログから最新のバイナリログまでを収集することができる。
さらに`--stop-never`と組み合わせることで、`Ctrl+C`などで停止するかマスタ側が停止するまで継続して収集する。
