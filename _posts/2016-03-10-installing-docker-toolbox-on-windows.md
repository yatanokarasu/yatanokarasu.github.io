---
layout:         post
title:          "Installing Docker Toolbox on Windows OS"
description:    "Docker Toolbox provides front-end client of docker in Windows OS."
keywords:       [Docker, Docker Toolbox]
categories:     [Virtualization]
tags:           [Docker, LXC, VirtualBox]
---

WindowsでDockerを使うには、VirtualBoxなどを使用してDocker Engineを動作できるLinux環境を準備する必要があります。
VirtualBoxでLinux環境を利用するだけであればVagrantでも問題ありませんが、
Docker Toolboxを利用することで、Dockerすぐに使えるLinux環境を手に入れることができます。

<div class="notice">
Dockerは64bit　OSでのみ動作します。
</div>

<!-- more -->

# Install Docker Toolbox

まずは[Docker Toolbox](https://www.docker.com/products/docker-toolbox)をダウンロードします。

ダウンロードしたらインストーラの案内に従って[Next > ]ボタンを押していきます。
{% include modal-image
    id="installer01"
    src="/images/posts/20160310/docker-toolbox-installer-01.png"
    class="half center"
%}

インストールパスはお好みで。
{% include modal-image
    id="installer02"
    src="/images/posts/20160310/docker-toolbox-installer-02.png"
    class="half center"
%}

Docker Tookboxでインストールされるコンポーネントを選択します。
私は、VirtualBoxとMsys2の環境を持っていたので、 __VirtualBox__ と __Git for Windows__ のチェックは外しました。
{% include modal-image
    id="installer03"
    src="/images/posts/20160310/docker-toolbox-installer-03.png"
    class="half center"
%}

ショートカット作成や`PATH`設定もお好みで。`PATH`設定しておいたほうが便利です。
{% include modal-image
    id="installer04"
    src="/images/posts/20160310/docker-toolbox-installer-04.png"
    class="half center"
%}

内容がOKであればインストールを開始します。
{% include modal-image
    id="installer05"
    src="/images/posts/20160310/docker-toolbox-installer-05.png"
    class="half center"
%}


# Usage Docker Toolbox

Windowsでは`docker-machine.exe`を使用します。
インストールフォルダ配下に`start.sh`があるので、Msys2など下記コマンドでdockerホストをVirtualBox上で起動することができます。

{% highlight raw %}
> bash --login start.sh
Running pre-create checks...
Creating machine...
(default) Copying C:\Users\${USER}\.docker\machine\cache\boot2docker.iso to C:\Users\${USER}\.docker\machine\machines\default\boot2docker.iso...
(default) Creating VirtualBox VM...
(default) Creating SSH key...
(default) Starting the VM...
(default) Check network to re-create if needed...
(default) Waiting for an IP...
Waiting for machine to be running, this may take a few minutes...
Detecting operating system of created instance...
Waiting for SSH to be available...
Detecting the provisioner...
Provisioning with boot2docker...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: C:\Program Files\tools\DockerToolbox\docker-machine.exe env default


                        ##         .
                  ## ## ##        ==
               ## ## ## ## ##    ===
           /"""""""""""""""""\___/ ===
      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~~ ~ /  ===- ~~~
           \______ o           __/
             \    \         __/
              \____\_______/

docker is configured to use the default machine with IP 192.168.99.100
For help getting started, check out the docs at https://docs.docker.com
{% endhighlight %}

ここまで表示されていれば、VirtualBox上で`default`というVM名でDockerホストが起動しています。


試しに、SSHでログインして、dockerの起動テストをしてみます。
SSHログインも`docker-machine.exe`からおこないます。

{% highlight raw %}
> docker-machine ssh default
                        ##         .
                  ## ## ##        ==
               ## ## ## ## ##    ===
           /"""""""""""""""""\___/ ===
      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~~ ~ /  ===- ~~~
           \______ o           __/
             \    \         __/
              \____\_______/
 _                 _   ____     _            _
| |__   ___   ___ | |_|___ \ __| | ___   ___| | _____ _ __
| '_ \ / _ \ / _ \| __| __) / _` |/ _ \ / __| |/ / _ \ '__|
| |_) | (_) | (_) | |_ / __/ (_| | (_) | (__|   <  __/ |
|_.__/ \___/ \___/ \__|_____\__,_|\___/ \___|_|\_\___|_|
Boot2Docker version 1.10.2, build master : 611be10 - Mon Feb 22 22:47:06 UTC 2016
Docker version 1.10.2, build c3959b1
docker@default:~$
{% endhighlight %}

これで、VirtualBox上で動いているDockerホストにSSHログインできました。
続いてdockerを動かしてみます。サブコマンドに`run`を指定し、コンテナ名と実行コマンドを指定します。
ここではコンテナ名を`centos`としていますが、ここれはデフォルトで提供されているもののようです。
初回は、Dockerリポジトリからコンテナイメージをダウンロードしてきますので少々時間がかかります。

{% highlight raw %}
docker@default:~$ docker run centos /bin/echo Hello World
Unable to find image 'centos:latest' locally
latest: Pulling from library/centos
a3ed95caeb02: Pull complete
196355c4b639: Pull complete
Digest: sha256:381f21e4c7b3724c6f420b2bcfa6e13e47ed155192869a2a04fa10f944c78476
Status: Downloaded newer image for centos:latest
Hello World
docker@default:~$
{% endhighlight %}

無事`Hello World`と`echo`表示させることができました。
