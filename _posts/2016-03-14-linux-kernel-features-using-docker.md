---
layout:         post
title:          "Linux Kernel features using Docker"
keywords:       ["Linux namespace", "CGroups", Docker]
categories:     [Infrastructure, Virtualization]
tags:           [Linux, Docker, "Reading book"]
---


Dockerに関する本を読んでます。

{% include modal-image
    id="book-cover"
    src="https://gihyo.jp/assets/images/gdp/2014/978-4-7741-6504-2.jpg"
    class="center half"
%}


Dockerを学んでいく中で、以下の機能が重要な要素であることを知ることができました。

*   Linux namespace
*   CGroups
*   UnionFS, AuFS, Brtfs


これらについて、少しずつ理解したことをまとめていこうと思います。

<!-- more -->


# What is Linux namespace?

プログラム言語とかでもよく聞く名前空間です。
DockerはLinux Kernelが提供してるこの仕組みを利用しています。
名前空間で隔離できるものは以下のようです。

MTN(Mount) Namespace
:   プロセスに見せるファイルシステムのマウント空間を分離できます。
    名前空間で隔離された中でマウントされたファイルシステムにファイルを書き込んでも、
    別の名前空間からは見えなくなるイメージです。
    
PID(Process ID) Namespace
:   プロセスが動作する空間を分離します。あらたに作られたプロセス空間からは、
    親空間で動作するプロセスが見えなくなります。逆に、親空間からは隔離されたプロセスも見えます。
    DockerホストからDockerコンテナ内のプロセスが見えても、Dockerコンテナ内からDockerホストの
    プロセスが見えないイメージです。

IPC(Inter-Process Communication) Namespace
:   メッセージキュー、セマフォ、共有メモリなど、Posixメッセージキューの空間を分離します。

UTS(Unix Time-sharing System) Namespace
:   unameシステムコールで取得できる情報を分離します。ホスト名とかNISドメインとかです。

NET(Network) Namespace
:   名前の通り、ネットワークデバイス、IPアドレスなどの情報を分離します。


Linuxの`linux-util`パッケージ内の`unshare`コマンドを使って名前空間での作業が試すことができます。



# What is CGroups?

CGroups(Control Groups)は、コンテナ内で使用するリソース(CPU、メモリ、I/O)を制御するために使用されます。

cpu
:   指定したプロセスが使用するCPU割合を指定できます

cpusets
:   何番目のCPUを使うかなど、指定したプロセスが動作するCPUを指定できます

memory
:   指定したプロセスが使用するメモリ量を指定できます

device
:   指定したプロセスが使用できるデバイスを指定できます



# What are UnionFS, AuFS and Btrfs?

UnionFSはLive CD(Knnopixとか)で使われているファイルシステムです。
書き込み不可能なKnnopix等で一時的にでもファイルの書き込みが可能なのは、
UnionFSがreadonlyのdisk imageの上に、あたかもファイルシステムがあるようにイメージを保持しているようです。
詳細はその他のサイトが詳しいのでGoogle画像検索をすると良いかもです。

AuFS(Another UnionFS)はLinuxファイルシステムに対するUnion FSみたいなもんらしいです。
Linux Kernelのメインラインではないみたいなので、今後使われないらしい。

Brtfs(B-Tree FS)は、B-Tree形式で何かしらの方法でファイルシステムを表現しているみたいです。


Dockerはこのような技術を駆使して、コンテナと呼ばれるプロセス空間を分離しているんですね。
逆に言えば、このような技術の特性上、ホスト型仮想化/ハイパーバイザ型仮想化などとは異なり、
ホストOSとは異なるOSは動作させることができないわけですね。

