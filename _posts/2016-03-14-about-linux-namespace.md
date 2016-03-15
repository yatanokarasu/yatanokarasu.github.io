---
layout:         post
title:          "About Linux namespace"
description:    "Linux namespace is used for separating resources such as process ID and is an important element of LXC (Linux Container)."
keywords:       ["Linux namespace", Docker]
categories:     [Infrastructure]
tags:           [Linux, "Linux namespace", LXC, "Reading book"]
---


Dockerに関する本を読んでます。

{% include modal-image
    id="book-cover"
    src="https://gihyo.jp/assets/images/gdp/2014/978-4-7741-6504-2.jpg"
    class="center half"
%}


Dockerを学んでいく中で、以下の機能が重要な要素であることを知ることができました。

*   Linux namespace
*   cgroup
*   Btrfs


これらについて、少しずつ理解したことをまとめていこうと思います。

<!-- more -->


# What is Linux namespace?

プログラム言語とかでもよく聞く名前空間です。
ここでの名前空間はLXC(Linux Container)を構成する重要な要素です。



