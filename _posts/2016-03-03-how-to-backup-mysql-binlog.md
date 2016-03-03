---
layout:     post
title:      How to backup MySQL binary logs
categories: [Infrastructure, Maintainance, Operation]
tags:       [MySQL, Backup/Restore]
keywords:   [MySQL, mysqlbinlog]
---


MySQLのバイナリログをバックアップする方法を考えていたところ、`mysqlbinlog`コマンドに`--read-from-remote-server`なるコマンドがあることに気付いた。
その他のオプションと組み合わせて使うことで、`scp`、`ssh+tar`、`rsync`などを使用しなくてもバックアップすることが可能になる。
さらに、あたかもレプリケーションスレーブの様にふるまい、マスター側でバイナリログが更新されたら都度バックアップすることも可能になる。

define
:   term1

define1
:   term2


*   item1

*   item2

    +   item2-1

<!-- more -->
